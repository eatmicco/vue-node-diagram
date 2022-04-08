<template>
  <div id="app">
    <diagram :x="0" :y="0" ref="diagram"/>
  </div>
</template>

<script>
import Diagram from 'v-node-diagram';

export default {
  name: 'App',
  data() {
    return {
      
    };
  },
  components: {
    Diagram
  },
  mounted() {
    console.log("App mounted!");
    this.$refs.diagram.clearNodes();
    this.$refs.diagram.addNode("node-3", 35, 35);
  }
}
</script>
