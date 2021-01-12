<template>
  <div id="app">
    <diagram :x="0" :y="0" />
  </div>
</template>

<script>
import Diagram from './components/Diagram.vue'

export default {
  name: 'App',
  components: {
    Diagram
  }
}
</script>

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
