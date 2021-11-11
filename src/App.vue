<template>
  <div id="app">
    <DatasetViewer></DatasetViewer>
  </div>
</template>

<script>
import DatasetViewer from "@/components/DatasetViewer";

export default {
  name: 'app',
  components: {
    DatasetViewer,
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
