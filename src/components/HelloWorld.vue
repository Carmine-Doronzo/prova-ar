<template>
  <div ref="arContainer" class="ar-container">
    <a-scene embedded arjs="sourceType: webcam; debugUIEnabled: false;" renderer="logarithmicDepthBuffer: true">
      <!-- Cubo che appare in un'area immaginaria -->
      <a-box position="0 0 -2" rotation="0 45 0" color="#4CC3D0" scale="0.5 0.5 0.5"></a-box>
      <a-entity camera></a-entity>
      <a-light type="directional" position="1 1 1"></a-light>
    </a-scene>
  </div>
</template>

<script>
import 'aframe';
import 'ar.js/aframe/build/aframe-ar.js'; // Importa AR.js come modulo

export default {
  name: 'ARComponent',
};
</script>

<style>
.ar-container {
  width: 100%;
  height: 100vh;
  background-color: #000;
}
</style>
