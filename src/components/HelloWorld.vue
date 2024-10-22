<template>
  <div ref="arContainer" class="ar-container"></div>
</template>

<script>
import * as THREE from 'three';
import { STLLoader } from 'three/examples/jsm/loaders/STLLoader.js';
import { ARButton } from 'three/examples/jsm/webxr/ARButton.js';

export default {
  name: 'ARComponent',
  mounted() {
    this.initAR();
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

      // Carica il modello STL
      const loader = new STLLoader();
      loader.load('path/to/your/model.stl', (geometry) => {
        const material = new THREE.MeshStandardMaterial({ color: 0x0055ff });
        this.mesh = new THREE.Mesh(geometry, material);

        // Riduci le dimensioni del modello
        this.mesh.scale.set(0.1, 0.1, 0.1); // Modifica i valori per scalare il modello

        this.mesh.visible = false; // Inizialmente nascosto
        scene.add(this.mesh);
      });

      // Imposta il rendering e la gestione degli eventi
      renderer.xr.enabled = true;
      const controller = renderer.xr.getController(0);
      scene.add(controller);

      // Evento per il tocco
      controller.addEventListener('selectstart', this.onSelectStart);
      controller.addEventListener('selectend', this.onSelectEnd);

      // Ciclo di rendering
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
        // Posiziona il modello alla posizione del controller
        this.mesh.position.copy(controller.position);
        this.mesh.visible = true; // Mostra il modello
      }
    },

    onSelectEnd(event) {
      // Gestisci l'evento di rilascio se necessario
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
</style>
