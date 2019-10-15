<template>
  <div id="app">
    <h1>rock-paper-scissors-lizard-spock (v1)</h1>
    <div id="instructions">
      <p><strong>Instructions</strong></p>
      <ol>
        <li>Select the appropriate gesture</li>
        <li>Click on the video to add the frozen frame to the set of images you wish to submit</li>
        <li>Add as many images as you like!</li>
        <li>Review the images to make sure they are correct (click on an image in the set below the video to remove it</li>
        <li>Click <em>Submit Training Data</em> when done!</li>
        <li>REPEAT! (Pretty please)</li>
      </ol>
      <p>Also, a couple of things to note:</p>
      <ul>
        <li>These images will be used to train a machine learning model for a keynote demo at Ignite</li>
        <li>Don't submit anything you don't wish others to see</li>
        <li><strong>PLEASE DO NOT SHARE THIS WITH ANYONE</strong></li>
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
