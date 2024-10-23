<template>
  <div ref="arContainer" class="ar-container">
    <a-scene embedded arjs>
      <a-marker preset="hiro">
        <!-- Modello STL -->
        <a-entity id="stlModel" :scale="modelScale" :position="modelPosition" :rotation="modelRotation"></a-entity>
        
        <!-- Cubo di test -->
        <a-box position="0 0.5 0" rotation="0 45 0" color="#4CC3D0" scale="0.5 0.5 0.5"></a-box>
      </a-marker>
      <a-entity camera></a-entity>
      <a-light type="directional" position="1 1 1"></a-light> <!-- Luce nella scena -->
    </a-scene>
  </div>
</template>

<script>
import 'aframe';
import 'ar.js/aframe/build/aframe-ar.js'; // Importa AR.js come modulo
import { STLLoader } from 'three/examples/jsm/loaders/STLLoader.js'; // Importa STLLoader

export default {
  name: 'ARComponent',
  data() {
    return {
      modelScale: '0.001 0.001 0.001', // Scala ridotta del modello
      modelPosition: '0 0 0', // Posizione centrale
      modelRotation: '0 0 0', // Rotazione iniziale
    };
  },
  mounted() {
    this.loadSTLModel('/path/to/your/model.stl'); // Percorso del tuo file STL
  },
  methods: {
    loadSTLModel(url) {
      const loader = new STLLoader();
      loader.load(url, (geometry) => {
        const material = new THREE.MeshStandardMaterial({ color: 0x0055ff });
        const mesh = new THREE.Mesh(geometry, material);
        mesh.scale.set(...this.modelScale.split(' ').map(Number)); // Scala il modello
        mesh.position.set(...this.modelPosition.split(' ').map(Number)); // Posizione del modello
        this.$refs.arContainer.appendChild(mesh); // Aggiungi il modello alla scena
      }, undefined, (error) => {
        console.error('Errore nel caricamento del modello:', error);
      });
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
