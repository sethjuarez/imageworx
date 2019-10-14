<template>
  <div id="app">
    <h1>rock-paper-scissors-lizard-spock (v1)</h1>
    <div id="instructions">
      <p><strong>Instructions</strong></p>
      <ol>
        <li>Select the appropriate gesture</li>
        <li>Click on the video to freeze the frame you think most adequately represents the gesture</li>
        <li>Click on the freezed frame to add to the set of images you wish to submit</li>
        <li>Repear as many times as you like!</li>
        <li>Click <em>Submit Training Data</em> when done!</li>
        <li>REPEAT! (Pretty please)</li>
      </ol>
      <p>Also, a couple of things to note:</p>
      <ul>
        <li>These images will be used to train a machine learning model for a keynote demo</li>
        <li>Don't submit anything you don't wish others to see</li>
      </ul>
    </div>
    <Capture/>
  </div>
</template>

<script>
import Capture from './components/Capture.vue'

export default {
  name: 'app',
  components: {
    Capture
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
#instructions {
  width: 640px;
  margin: 0px auto;
  text-align: left;
}
</style>
