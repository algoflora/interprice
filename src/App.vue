<template>
  <div id="app">
    <inter-price :data="$options.dataJson"/>
  </div>
</template>

<script>
import InterPrice from './components/InterPrice.vue'
import DATA_JSON from './assets/data.json'

export default {
  name: 'App',
  dataJson: DATA_JSON,
  components: {
    InterPrice
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
