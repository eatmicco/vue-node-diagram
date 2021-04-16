<template>
  <div id="app">
    <diagram :x="0" :y="0" />
  </div>
</template>

<script>
import Diagram from 'v-node-diagram';

export default {
  name: 'App',
  components: {
    Diagram
  }
}
</script>
