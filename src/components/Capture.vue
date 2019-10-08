<template>
    <div>
        <video v-on:click="freeze()" id="video" width="320" height="240" autoplay></video>
        <canvas id="canvas" width="320" height="240"></canvas>
    </div>
</template>

<script>
    export default {
        name: 'Capture',
        data: {
            video: null,
            canvas: null
        },
        created: function () {
            
        },
        mounted: async function () {
            let canvas = document.getElementById('canvas');
            this.canvas = canvas.getContext('2d');
            this.video = document.getElementById('video')
            if(navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                let stream = await navigator.mediaDevices.getUserMedia({ video: true })
                this.video.srcObject = stream
                this.vide.play()
            }
        },
        methods: {
            freeze: function () {
                this.canvas.drawImage(this.video, 0, 0, 320, 240);
            }
        }
    }
</script>

<style scoped>
    video, canvas {
        border: solid 1px gray;
    }
</style>