<template>
  <div id="app">
    <h1>rock-paper-scissors-lizard-spock (v2)</h1>
    <FastCapture/>
  </div>
</template>

<script>
import FastCapture from './components/FastCapture.vue'

export default {
  name: 'app',
  components: {
    FastCapture
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
