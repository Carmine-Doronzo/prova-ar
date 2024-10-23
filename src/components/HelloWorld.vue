<template>
  <div ref="arContainer" class="ar-container"></div>
  <div v-if="!isMobile" ref="canvasContainer" class="canvas-container"></div>
</template>

<script>
import * as THREE from 'three';
import { STLLoader } from 'three/examples/jsm/loaders/STLLoader.js';
import { ARButton } from 'three/examples/jsm/webxr/ARButton.js';

export default {
  name: 'ARComponent',
  data() {
    return {
      mesh: null,
      isSqueezing: false,
      initialPosition: null,
      initialRotation: null,
      isMobile: false,
    };
  },
  mounted() {
    this.isMobile = this.checkIfMobile();
    
    if (window.navigator.xr) {
      this.initAR(); // Usa WebXR per browser compatibili
    } else if (this.isIOS()) {
      this.showARQuickLook(); // Fallback per Safari su iOS
    } else {
      this.init3DCanvas(); // Visualizza modello 3D nel canvas se non è un dispositivo mobile
    }
  },
  methods: {
    checkIfMobile() {
      return /Android|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
    },
    isIOS() {
      return /iPad|iPhone|iPod/.test(navigator.userAgent) && !window.MSStream;
    },
    showARQuickLook() {
      const usdzLink = '/skull_mug.usdz'; // Percorso del file .usdz
      const anchor = document.createElement('a');
      anchor.setAttribute('rel', 'ar');
      anchor.setAttribute('href', usdzLink);
      anchor.innerHTML = `<img src="${usdzLink}" alt="Visualizza in AR" style="display:none;">`;
      document.body.appendChild(anchor);
      anchor.click();
    },
    
    initAR() {
      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
      
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(ARButton.createButton(renderer));
      this.$refs.arContainer.appendChild(renderer.domElement);
      
      const light = new THREE.DirectionalLight(0xffffff, 1);
      light.position.set(1, 1, 1).normalize();
      scene.add(light);

      const loader = new STLLoader();
      loader.load('/skull_mug.stl', (geometry) => {
        const material = new THREE.MeshStandardMaterial({ color: 0x0055ff });
        this.mesh = new THREE.Mesh(geometry, material);
        this.mesh.scale.set(0.001, 0.001, 0.001);
        this.mesh.visible = false;
        this.mesh.position.set(0, 0, -10);
        scene.add(this.mesh);
      });

      renderer.xr.enabled = true;
      const controller = renderer.xr.getController(0);
      scene.add(controller);

      controller.addEventListener('selectstart', this.onSelectStart);
      controller.addEventListener('selectend', this.onSelectEnd);
      controller.addEventListener('squeezestart', this.onSqueezeStart);
      controller.addEventListener('squeezeend', this.onSqueezeEnd);

      const animate = () => {
        renderer.setAnimationLoop(() => {
          if (this.isSqueezing) {
            this.updateModelScale();
            this.updateModelPosition(controller);
            this.updateModelRotation(controller);
          }
          renderer.render(scene, camera);
        });
      };
      animate();
    },
    
    init3DCanvas() {
      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      const renderer = new THREE.WebGLRenderer({ antialias: true });
      
      renderer.setSize(window.innerWidth, window.innerHeight);
      this.$refs.canvasContainer.appendChild(renderer.domElement);

      const light = new THREE.DirectionalLight(0xffffff, 1);
      light.position.set(1, 1, 1).normalize();
      scene.add(light);

      const loader = new STLLoader();
      loader.load('/skull_mug.stl', (geometry) => {
        const material = new THREE.MeshStandardMaterial({ color: 0x0055ff });
        this.mesh = new THREE.Mesh(geometry, material);
        this.mesh.scale.set(0.01, 0.01, 0.01); // Scala più grande per il canvas 3D
        scene.add(this.mesh);
      });

      const animate = () => {
        requestAnimationFrame(animate);
        this.mesh.rotation.x += 0.01;
        this.mesh.rotation.y += 0.01;
        renderer.render(scene, camera);
      };
      animate();
    },
    
    // Altri metodi come onSelectStart, updateModelPosition, updateModelRotation, ecc.
  },
};
</script>


<style>
.ar-container {
  width: 100%;
  height: 100vh;
  background-color: #000;
  position: relative;
}

.canvas-container {
  width: 100%;
  height: 100vh;
  background-color: #000;
  position: relative;
}

.ar-container a[rel="ar"] {
  display: block;
  width: 100%;
  height: 100%;
  background-color: #fff;
}
</style>
