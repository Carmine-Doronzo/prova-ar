<template>
  <div ref="arContainer" class="ar-container"></div>
</template>

<script>
import { MindAR } from 'mind-ar';
import * as THREE from 'three';
import { STLLoader } from 'three/examples/jsm/loaders/STLLoader.js';

export default {
  name: 'ARComponent',
  data() {
    return {
      mesh: null,
      arApp: null,
      // Definisci i parametri dell'area delimitata
      areaWidth: 2, // Larghezza dell'area
      areaHeight: 2, // Altezza dell'area
    };
  },
  mounted() {
    this.initAR();
  },
  methods: {
    async initAR() {
      // Crea un'applicazione MindAR
      this.arApp = new MindAR({
        container: this.$refs.arContainer,
        // Utilizza un sistema marker-less
        onTracking: (state) => {
          if (state.tracking) {
            this.mesh.visible = true; // Rendi visibile il modello quando il tracciamento è attivo
          } else {
            this.mesh.visible = false; // Nascondi il modello quando il tracciamento non è attivo
          }
        }
      });

      // Carica il modello STL
      const loader = new STLLoader();
      loader.load('/skull_mug.stl', (geometry) => {
        const material = new THREE.MeshStandardMaterial({ color: 0x0055ff });
        this.mesh = new THREE.Mesh(geometry, material);
        this.mesh.scale.set(0.001, 0.001, 0.001);
        this.mesh.position.set(0, 0, -0.5); // Posiziona il modello nell'area delimitata
        this.arApp.scene.add(this.mesh); // Aggiungi il modello alla scena MindAR
      });

      // Avvia l'app MindAR
      await this.arApp.start();
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
