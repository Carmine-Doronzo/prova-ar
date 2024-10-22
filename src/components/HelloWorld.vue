<template>
  <div ref="arContainer" class="ar-container"></div>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { ARButton } from 'three/examples/jsm/webxr/ARButton.js';

export default {
  name: 'ARComponent',
  mounted() {
    this.initAR();
  },
  data() {
    return {
      isDragging: false, // Flag per tenere traccia dello stato di trascinamento
      mesh: null, // Il modello 3D caricato
    };
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

      // Carica il modello GLB
      const loader = new GLTFLoader();
      loader.load('/watchstand_newV.glb', (gltf) => {
        this.mesh = gltf.scene; // Il modello caricato è contenuto in gltf.scene

        // Riduci le dimensioni del modello alla grandezza di una bottiglia
        this.mesh.scale.set(0.01, 0.01, 0.01); // Dimensione più piccola simile a una bottiglia
        this.mesh.visible = false; // Inizialmente nascosto

        // Posiziona il modello lontano dalla telecamera
        this.mesh.position.set(0, 0, 25); // Posizione iniziale a 5 metri dalla telecamera
        scene.add(this.mesh);
      }, undefined, (error) => {
        console.error('Errore nel caricamento del modello GLB:', error);
      });

      // Imposta il rendering e la gestione degli eventi
      renderer.xr.enabled = true;
      const controller = renderer.xr.getController(0);
      scene.add(controller);

      // Evento per il tocco
      controller.addEventListener('selectstart', this.onSelectStart);
      controller.addEventListener('selectend', this.onSelectEnd);
      controller.addEventListener('selectmove', this.onSelectMove); // Aggiunto per il movimento

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
        this.isDragging = true; // Inizia il trascinamento
      }
    },

    onSelectEnd(event) {
      this.isDragging = false; // Ferma il trascinamento
    },

    onSelectMove(event) {
      if (this.isDragging && this.mesh) {
        const controller = event.target;

        // Calcola la nuova posizione del modello in base alla posizione del controller
        this.mesh.position.copy(controller.position);
        
        // Limita il movimento dell'oggetto a un'area di 1000 metri
        const maxDistance = 1000; // Distanza massima in metri
        this.mesh.position.x = THREE.MathUtils.clamp(this.mesh.position.x, -maxDistance, maxDistance);
        this.mesh.position.y = THREE.MathUtils.clamp(this.mesh.position.y, -maxDistance, maxDistance);
        this.mesh.position.z = THREE.MathUtils.clamp(this.mesh.position.z, -maxDistance, maxDistance);
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
</style>
