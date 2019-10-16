<template>
    <div v-on:keydown="key">
        <div id="instructions">
            <p><strong>Fast Capture Instructions</strong></p>
            <ol>
                <li>Select desired gesture</li>
                <li>Get into position!</li>
                <li>Spacebar to start</li>
            </ol>
            <p>Also, a couple of things to note:</p>
            <ul>
                <li>These images will be used to train a machine learning model for a keynote demo at Ignite</li>
                <li>Don't submit anything you don't wish others to see</li>
                <li><strong>PLEASE DO NOT SHARE THIS WITH ANYONE</strong></li>
            </ul>
        </div>
        <div id="radioselection">
            <span :key="item+index" v-for="(item, index) in signs">
                <input type="radio" :id="item" name="sign" :value="item" 
                    :checked="selectedSign == item"
                    v-model="selectedSign">
                <label :for="item">{{item}}</label>
                &nbsp;
            </span>
            <div v-if="interval != null">Click Spacebar to Stop!</div>
        </div>
        <div>
            <video id="video" width="320" height="240" autoplay></video>
            <canvas id="canvas" width="320" height="240"></canvas>
        </div>
        <div id="listOPics" v-if="list.length > 0">
            <div>Click on an image to remove (or <button type="button" v-on:click="clearImages()">Clear All</button>)</div>
            <div>(Also maybe clean out any images that might be ambiguous if possible)</div>
            <ul class="imagelist" :key="index" v-for="(item, index) in list">
                <li class="imgitem" @click="removeImage(index)">
                    <div>{{item.type}}</div>
                    <img height="120" width="160" :src="item.image" />
                </li>
            </ul>
            <div class="btn">
                <button type="button" v-if="list.length > 0" v-on:click="submitImages()" v-show="!processing">Submit Training Data</button>
            </div>
        </div>
        <div id="notifications" v-show="processing">
            {{message}}
        </div>
    </div>
</template>

<script>
    import axios from 'axios'
    export default {
        name: 'FastCapture',
        data: function () {
            return {
                processing: false,
                message: '',
                video: null,
                canvas: null,
                signs: ['rock', 'paper', 'scissors', 'lizard', 'spock'],
                selectedSign: 'rock',
                list: [],
                interval: null
            }
        },
        mounted: async function () {
            let canvas = document.getElementById('canvas');
            this.canvas = canvas.getContext('2d');
            this.video = document.getElementById('video')
            if(navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                let stream = await navigator.mediaDevices.getUserMedia({ video: true })
                this.video.srcObject = stream
                this.video.play()
            }
        },
        methods: {
            key: function (event) {
                if(event.keyCode == 32) {
                    if(this.interval != null)
                        this.stopCapture()
                    else
                        this.startCapture()
                }
            },
            startCapture: function () {
                this.stopCapture()
                setTimeout(this.stopCapture, 60010);
                this.interval = setInterval(this.addImage, 500)
                this.video.style.border = "thick solid #FF0000";
            },
            stopCapture: function () {
                if(this.interval != null) {
                    clearInterval(this.interval);
                    this.interval = null;
                    this.video.style.border = "solid 1px gray";
                }
            },
            addImage: function () {
                this.canvas.drawImage(this.video, 0, 0, 320, 240);
                let c = document.getElementById('canvas')
                this.list.push({
                    type: this.selectedSign,
                    image: c.toDataURL()
                })
            },
            removeImage: function (index) {
                this.list.splice(index, 1)
            },
            clearImages: function () {
                this.list = [];
            },
            submitImages: async function () {
                this.processing = true
                this.message = 'sending data'
                let url = 'https://imageworxapi.azurewebsites.net/api/save'
                let response = await axios.post(url, { items: this.list }, {
                    headers: { 'Content-Type': 'application/json' }
                })
                this.message = 'done!'
                this.list = []
                let data = response['data']
                console.log(data)
                this.processing = false
            }
        }
    }
</script>

<style scoped>
    #instructions {
        width: 640px;
        margin: 0px auto;
        text-align: left;
    }
    video {
        border: solid 1px gray;
    }

    canvas {
        display: none;
    }

    #listOPics {
        clear: both;
        padding: 25px 100px;
        text-align: center;
        margin: 0px;
    }
    #radioselection {
        margin-bottom: 10px;
    }

    #notifications {
        width:150px;
        height:30px;
        display:table-cell;
        text-align:center;
        background:rgb(255, 166, 0);
        border:1px solid #000;
        bottom: 10px;
        right: 10px;
        position: absolute;
        padding-top: 10px;
    }

    .imagelist {
        list-style-type: none;
        padding: 0px;
    }

    .imgitem {
        float: left;
        padding: 10px;
    }

    .btn {
        text-align: center;
        clear: both;
    }
</style>