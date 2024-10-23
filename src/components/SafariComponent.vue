<template>
    <div ref="arContainer" class="ar-container">
      <a href="javascript:void(0)" @click="openAR" v-if="isSafari">
        apri AR
      </a>
    </div>
  </template>
  
  <script>
  export default {
    name: 'ARComponent',
    data() {
      return {
        modelUrl: '/skull_mug.usdz', // Assicurati di avere il tuo modello in formato USDZ
        isSafari: false,
      };
    },
    mounted() {
      this.isSafari = this.checkSafari();
      if (!this.isSafari) {
        console.warn('AR non supportato in questo browser. Utilizza Safari.');
      }
    },
    methods: {
      checkSafari() {
        return /^((?!chrome|android).)*safari/i.test(navigator.userAgent);
      },
  
      openAR() {
        const anchor = document.createElement('a-entity');
        anchor.setAttribute('href', this.modelUrl);
        anchor.setAttribute('rel', 'ar');
        anchor.setAttribute('style', 'display: none;');
  
        // Posizionamento: 50 cm dalla telecamera e dimensioni dell'area 50 cm x 50 cm
        anchor.setAttribute('position', '0 0 -0.5');
        anchor.setAttribute('scale', '0.1 0.1 0.1'); // Dimensioni in scala per 50 cm
  
        document.body.appendChild(anchor);
        anchor.click(); // Simula un clic per aprire AR Quick Look
        document.body.removeChild(anchor); // Rimuove l'ancora dopo l'uso
      },
    },
  };
  </script>
  
  <style>
  .ar-container {
    width: 100%;
    height: 30vh;
    background-color: #000;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  </style>
  