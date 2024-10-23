<template>
  <div ref="arContainer1" class="ar-container1"></div>
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
    };
  },
  mounted() {
    if (window.navigator.xr) {
      this.initAR(); // Utilizza WebXR per i browser compatibili
    } else if (this.isIOS()) {
      this.showARQuickLook(); // Fallback per Safari iOS
    } else {
      console.console.warn
      ('AR non supportato su questo browser.');
    }
  },
  methods: {
    isIOS() {
      return /iPad|iPhone|iPod/.test(navigator.userAgent) && !window.MSStream;
    },
    showARQuickLook() {
      const usdzLink = '/skull-mug.usdz'; // Percorso del file .usdz
      const anchor = document.createElement('a');
      anchor.setAttribute('rel', 'ar');
      anchor.setAttribute('href', usdzLink);
      anchor.innerHTML = `<img src="${usdzLink}" alt="Visualizza in AR" style="display:none;">`;
      document.body.appendChild(anchor);
      anchor.click(); // Simula un clic per avviare l'AR
    },
    
    initAR() {
      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });

      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(ARButton.createButton(renderer));
      this.$refs.arContainer1.appendChild(renderer.domElement);

      const light = new THREE.DirectionalLight(0xffffff, 1);
      light.position.set(1, 1, 1).normalize();
      scene.add(light);

      const loader = new STLLoader();
      loader.load('/watchstand_newV.stl', (geometry) => {
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
    
    // Altri metodi come onSelectStart, updateModelPosition, updateModelRotation, ecc.
  },
};
</script>

<style>
.ar-container1 {
  width: 100%;
  height: 30vh;
  background-color: #000;
  position: relative;
}

.ar-container1 a[rel="ar"] {
  display: block;
  width: 100%;
  height: 100%;
  background-color: #fff;
}
</style>
