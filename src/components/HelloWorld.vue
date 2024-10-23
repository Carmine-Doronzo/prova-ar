<template>
  <div ref="arContainer" class="ar-container"></div>
  
  <!-- Controlli visivi e istruzioni -->
  <div class="controls">
    <p>Tocca lo schermo per posizionare il modello.</p>
    <button @click="rotateModel">Ruota</button>
    <button @click="scaleUpModel">Ingrandisci</button>
    <button @click="scaleDownModel">Riduci</button>
  </div>
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
      isRotating: false, // Flag per rotazione
      scaleFactor: 1, // Fattore di scala
    };
  },
  mounted() {
    if (navigator.xr) {
      // Chrome (Android) e altri browser con supporto WebXR
      this.initAR();
    } else if (this.isIOS()) {
      // Safari (iOS) con AR Quick Look
      this.showARQuickLook();
    } else {
      alert('AR non supportato su questo dispositivo o browser.');
    }
  },
  methods: {
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
      anchor.click(); // Simula un clic per avviare AR Quick Look
    },
    initAR() {
      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });

      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // Migliora la compatibilità su Android

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
        this.mesh.scale.set(0.001, 0.001, 0.001); // Dimensioni ridotte
        this.mesh.position.set(0, 0, -0.5); // Posizionamento iniziale
        this.mesh.visible = false; // Inizialmente nascosto
        scene.add(this.mesh);
      });

      renderer.xr.enabled = true;

      // Aggiungi controller WebXR e gestisci eventi
      const controller = renderer.xr.getController(0);
      scene.add(controller);

      controller.addEventListener('selectstart', (event) => {
        console.log('Select start', event.target.position);
        this.onSelectStart(event);
      });

      // Ciclo di animazione
      const animate = () => {
        renderer.setAnimationLoop(() => {
          if (this.mesh && this.mesh.visible) {
            if (this.isRotating) {
              this.mesh.rotation.y += 0.05; // Rotazione automatica
            }
            this.mesh.scale.set(this.scaleFactor, this.scaleFactor, this.scaleFactor); // Scala il modello
          }
          renderer.render(scene, camera);
        });
      };
      animate();
    },
    onSelectStart(event) {
      const controller = event.target;
      if (this.mesh) {
        // Posiziona il modello vicino al controller
        this.mesh.position.copy(controller.position);
        this.mesh.position.z -= 0.5; // Posizionalo leggermente davanti al controller
        this.mesh.visible = true; // Mostra il modello
        this.mesh.updateMatrix(); // Forza l'aggiornamento della matrice
      }
    },
    rotateModel() {
      // Alterna la rotazione
      this.isRotating = !this.isRotating;
    },
    scaleUpModel() {
      // Aumenta il fattore di scala
      this.scaleFactor += 0.1;
    },
    scaleDownModel() {
      // Riduci il fattore di scala
      if (this.scaleFactor > 0.1) {
        this.scaleFactor -= 0.1;
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

.controls {
  position: absolute;
  top: 10px;
  left: 10px;
  background-color: rgba(255, 255, 255, 0.8);
  padding: 10px;
  border-radius: 8px;
}

.controls p {
  margin: 0 0 10px;
  font-size: 14px;
}

.controls button {
  display: block;
  margin-bottom: 5px;
  padding: 8px 12px;
  font-size: 14px;
  cursor: pointer;
}
</style>
