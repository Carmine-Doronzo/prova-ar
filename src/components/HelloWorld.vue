<template>
  <div ref="arContainer" class="ar-container"></div>
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
    };
  },
  mounted() {
    if (navigator.xr) {
      // Chrome (Android) e altri browser con supporto WebXR
      this.initAR();
    } else if (this.isIOS()) {
      // Safari (iOS) con fallback su AR Quick Look
      this.showARQuickLook();
    } else {
      alert('AR non supportato su questo dispositivo o browser.');
    }
  },
  methods: {
    // Rileva se l'utente è su iOS
    isIOS() {
      return /iPad|iPhone|iPod/.test(navigator.userAgent) && !window.MSStream;
    },
    // Fallback per Safari (iOS) con AR Quick Look
    showARQuickLook() {
      const usdzLink = '/skull_mug.usdz'; // Percorso del file .usdz
      const anchor = document.createElement('a');
      anchor.setAttribute('rel', 'ar');
      anchor.setAttribute('href', usdzLink);
      anchor.innerHTML = `<img src="${usdzLink}" alt="Visualizza in AR" style="display:none;">`;
      document.body.appendChild(anchor);
      anchor.click(); // Simula un clic per avviare AR Quick Look
    },
    // Inizializza WebXR per i browser compatibili
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

      // Carica il modello STL
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
          renderer.render(scene, camera);
        });
      };
      animate();
    },
    onSelectStart(event) {
      const controller = event.target;
      if (this.mesh) {
        this.mesh.position.copy(controller.position);
        this.mesh.visible = true;
      }
    },
    onSqueezeStart() {
      this.isSqueezing = true;
    },
    onSqueezeEnd() {
      this.isSqueezing = false;
    },
    updateModelScale() {
      if (this.isSqueezing) {
        this.mesh.scale.multiplyScalar(0.05);
      } else {
        this.mesh.scale.multiplyScalar(0.1);
      }
    },
  },
};
</script>

<style>
.ar-container {
  width: 100%;
  height: 100vh;
  background-color: #000;
}

.ar-container a[rel="ar"] {
  display: block;
  width: 100%;
  height: 100%;
  background-color: #fff;
}
</style>
