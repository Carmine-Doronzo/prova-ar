
<template>
  <div>
    <a-scene embedded arjs>
      <!-- Posizioniamo il marker a 200 metri di distanza rispetto alla camera -->
      <a-marker preset="/pattern-marker.patt" position="0 0 -200">
        <a-entity 
          gltf-model="/watchstand_newV.glb" 
          scale="0.005 0.005 0.005" 
          position="0 0 0"> <!-- Posizione relativa al marker -->
        </a-entity>
      </a-marker>
      <!-- <a-entity camera position="0 0 0"></a-entity> -->
    </a-scene>
  </div>
</template>

<script>
export default {
  name: 'ARComponent',
};
</script>

<style>
html, body {
  margin: 0;
  overflow: hidden;
}
a-scene {
  width: 100%;
  height: 100vh;
}
</style>


