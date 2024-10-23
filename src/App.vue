<script>
import HelloWorld from './components/HelloWorld.vue'
import safaricomponent from './components/safaricomponent.vue'
export default{
  components: {
    HelloWorld,
    safaricomponent
  },
}
</script>

<template>
  <h1>safari</h1>
  <safaricomponent/>
  <h1>chrome</h1>
  <HelloWorld/>
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
