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
    // this.initAR()
    if (this.isChromeOnAndroid()) {
      this.initAR(); // Utilizza WebXR per i browser compatibili
    } else if (this.isIOS()) {
      this.showARQuickLook(); // Fallback per Safari iOS
    } else {
      console.warn
        ('AR non supportato su questo browser.');
    }
  },
  methods: {
    isIOS() {
      return /iPad|iPhone|iPod/.test(navigator.userAgent) && !window.MSStream;
    },
    isChromeOnAndroid() {
      const userAgent = navigator.userAgent.toLowerCase();
      const isAndroid = userAgent.includes("android");
      const isChrome = userAgent.includes("chrome") && !userAgent.includes("edg"); // Esclude Edge che può usare lo stesso user agent
      return isAndroid && isChrome;
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
      this.$refs.arContainer.appendChild(renderer.domElement);

      const light = new THREE.DirectionalLight(0xffffff, 1);
      light.position.set(1, 1, 1).normalize();
      scene.add(light);

      // Carica il modello STL
      const loader = new STLLoader();
      loader.load('/skull_mug.stl', (geometry) => {
        const material = new THREE.MeshStandardMaterial({ color: 0x0055ff });
        this.mesh = new THREE.Mesh(geometry, material);

        // Riduci le dimensioni del modello
        this.mesh.scale.set(0.001, 0.001, 0.001); // Dimensione iniziale
        this.mesh.visible = false; // Inizialmente nascosto
        this.mesh.position.set(0, 0, -10);
        scene.add(this.mesh);
      });

      // Imposta il rendering e la gestione degli eventi
      renderer.xr.enabled = true;
      const controller = renderer.xr.getController(0);
      scene.add(controller);

      // Evento per il tocco
      controller.addEventListener('selectstart', this.onSelectStart);
      controller.addEventListener('selectend', this.onSelectEnd);
      controller.addEventListener('squeezestart', this.onSqueezeStart); // Gestisce l'inizio della pressione
      controller.addEventListener('squeezeend', this.onSqueezeEnd); // Gestisce la fine della pressione

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

    onSqueezeStart(event) {
      this.isSqueezing = true; // Flag per tenere traccia dello stato di pressione
    },

    onSqueezeEnd(event) {
      this.isSqueezing = false; // Ripristina il flag
    },

    updateModelScale() {
      if (this.isSqueezing) {
        // Avvicina il modello
        this.mesh.scale.multiplyScalar(0.05); // Riduci la scala del modello
      } else {
        // Allontana il modello
        this.mesh.scale.multiplyScalar(0.1); // Aumenta la scala del modello
      }
    },

    animate() {
      // Ciclo di rendering
      const animate = () => {
        requestAnimationFrame(animate);
        if (this.isSqueezing) {
          this.updateModelScale(); // Aggiorna la scala mentre si preme
        }
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

.ar-container a[rel="ar"] {
  display: block;
  width: 100%;
  height: 100%;
  background-color: #fff;
}
</style>
