<template>
    <div>
        <div id="instructions">
            <p><strong>Fast Capture Instructions</strong></p>
            <ol>
                <li>Select desired gesture</li>
                <li>Get into position!</li>
                <li>Spacebar to start</li>
            </ol>
            <p>Also, a couple of things to note:</p>
            <ul>
                <li>Max images that can be submitted at a time is 64 (Current count is at {{list.length}})</li>
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
        <div id="images">
            <video @click="predict()" id="video" width="320" height="240" autoplay></video>
            <canvas id="rendered" width="224" height="224"></canvas>
            <canvas id="canvas" width="320" height="240"></canvas>
            <div id="output">
                <div id="flavor" v-if="modelmeta != null">Flavor: <strong>{{modelmeta.Flavor}}</strong></div>
                <div id="exported" v-if="modelmeta != null">Exported: {{modelmeta.ExportedDate}}</div>
                <div id="prediction">{{guess}}</div>
                <div id="plist">
                    <ul :key="idx" v-for="(pitem, idx) in probabilities">
                        <li>{{pitem.label}}: {{pitem.probability.toFixed(2)}}%</li>
                    </ul>
                </div>
            </div>
        </div>
        <div id="listOPics" v-if="list.length > 0">
            <div>Click on an image to remove (or <button type="button" v-on:click="clearImages()">Clear All</button>)</div>
            <div>(Also maybe clean out any images that might be ambiguous if possible)</div>
            <div id="warning" v-if="list.length >= 64">64 Limit Reached - either submit or remove some images</div>
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
    import $ from 'jquery'
    import * as tf from '@tensorflow/tfjs';

    export default {
        name: 'FastCapture',
        data: function () {
            return {
                processing: false,
                message: '',
                video: null,
                canvas: null,
                signs: ['none', 'rock', 'paper', 'scissors', 'lizard', 'spock'],
                selectedSign: 'none',
                list: [],
                lastresponse: null,
                interval: null,
                model: null,
                modelmeta: null,
                labels: [],
                probabilities: [],
                guess: null,
                vdim: {
                    'width': 0,
                    'height': 0
                }
            }
        },
        mounted: async function () {
            // map spacebar key event
            document.onkeyup = this.key
            // get canvas context
            let canvas = document.getElementById('canvas')
            this.canvas = canvas.getContext('2d')
            // run and start video
            this.video = document.getElementById('video')
            if(navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                let stream = await navigator.mediaDevices.getUserMedia({ video: true })
                let tracks = stream.getVideoTracks()
                if(tracks.length >= 1) {
                    let settings = tracks[0].getSettings()
                    this.vdim.width = settings.width
                    this.vdim.height = settings.height
                    this.video.srcObject = stream
                    this.video.play()
                }
            }

            // load model and metadata
            this.model = await tf.loadGraphModel('model/model.json')
            this.modelmeta = await $.getJSON('model/cvexport.manifest')
            const l = await $.get('model/labels.txt')
            this.labels = l.split('\r\n')

            // start prediction run
            setInterval(this.predict, 250)
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
                setTimeout(this.stopCapture, 60010)
                this.interval = setInterval(this.addImage, 500)
                this.video.style.border = "thick solid #FF0000"
            },
            stopCapture: function () {
                if(this.interval != null) {
                    clearInterval(this.interval);
                    this.interval = null;
                    this.video.style.border = "solid 1px gray"
                }
            },
            addImage: function () {
                if(this.list.length <  64) {
                    this.canvas.drawImage(this.video, 0, 0, 320, 240)
                    let c = document.getElementById('canvas')
                    this.list.push({
                        type: this.selectedSign,
                        image: c.toDataURL()
                    })
                } else {
                    // reached max
                    this.stopCapture()
                }
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
                this.lastresponse = response['data']
                this.processing = false
            },
            predict: async function () {
                var pic = document.getElementById('rendered')
                var ctx = pic.getContext('2d')

                // clip out edges to make square
                let xclip = (this.vdim.width - this.vdim.height) / 2
                ctx.drawImage(this.video, xclip, 0, this.vdim.height, this.vdim.height, 0, 0, 224, 224)
                var img = ctx.getImageData(0, 0, 224, 224).data

                var imagedata = []
                for(var i = 0; i < img.length; i += 4) {
                    // [RGBA] -> [BGR]
                    // 1. Discard img[i+3] - Alpha channel
                    // 2. Swap R/G
                    imagedata.push(img[i+2], img[i+1], img[i])
                }

                var tensor = tf.tensor1d(imagedata).reshape([-1, 224, 224, 3])
                
                var pred =  await this.model
                                        .predict({'Placeholder': tensor}, {'batchSize': 1})
                                        .reshape([6])
                                        .data()
                
                this.guess = this.labels[pred.indexOf(Math.max(...pred))]
                this.probabilities = []
                for(var j = 0; j < pred.length; j++)
                    this.probabilities.push({ 'label': this.labels[j], 'probability': pred[j]*100 })
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
        transform: rotateY(180deg);
        -webkit-transform:rotateY(180deg); /* Safari and Chrome */
        -moz-transform:rotateY(180deg); /* Firefox */
        float: left;
    }

    #output {
        border: solid 1px gray;
        height: 240px;
        width: 320px;
        margin-left: 10px;
        float: left;
        clear: right;
        text-align: left;
        padding: 0px 4px;
    }

    #images{
        margin: 0px auto;
        width: 700px;
        /* border: solid 10px red;*/
    }

    #rendered {
        display: none;
    }

    #canvas {
        display: none;
    }
    #warning {
        color: red;
        font-size: 16px;
        font-weight: bold;
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
    #plist ul {
        margin: 1px;
    }
    #plist li {
        /*border: solid 1px black;*/
        margin: 5px;
    }

    #prediction {
        text-align: center;
        margin: 3px;
        font-size: 30px;
        color: red;
        font-weight: bolder;
    }

    #flavor {
        margin-top: 5px;
        margin-bottom: 5px;
    }

    #exported {
        margin-bottom: 5px;
    }
</style>