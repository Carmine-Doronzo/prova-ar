<script>
import HelloWorld from './components/HelloWorld.vue'
import SafariComponent from './components/SafariComponent.vue'
export default{
  components: {
    HelloWorld,
    SafariComponent
  },
}
</script>

<template>
  <!-- <h1>safari</h1> -->
  <SafariComponent/>
  <!-- <h1>chrome</h1>
  <HelloWorld/> -->
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
