<template>
  <div id="app">
    <h1>rock-paper-scissors-lizard-spock (v1)</h1>
    <FastCapture/>
  </div>
</template>

<script>
import Capture from './components/Capture.vue'
import FastCapture from './components/FastCapture.vue'

export default {
  name: 'app',
  components: {
    Capture,
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
