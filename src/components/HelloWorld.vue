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
    this.initAR(); // Inizializza AR senza AR Quick Look
  },
  methods: {
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
        this.mesh.position.set(0, 0, -1);
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

    onSelectStart() {
      if (this.mesh) {
        this.mesh.visible = true; // Rende il modello visibile quando selezionato
      }
    },
    
    onSelectEnd() {
      if (this.mesh) {
        this.mesh.visible = false; // Nasconde il modello quando deselezionato
      }
    },
    
    onSqueezeStart() {
      this.isSqueezing = true;
    },

    onSqueezeEnd() {
      this.isSqueezing = false;
    },

    updateModelScale() {
      // Implementa la logica per aggiornare la scala del modello qui
    },

    updateModelPosition(controller) {
      // Implementa la logica per aggiornare la posizione del modello qui
    },

    updateModelRotation(controller) {
      // Implementa la logica per aggiornare la rotazione del modello qui
    },
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
</style>
