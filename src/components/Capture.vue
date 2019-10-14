<template>
    <div>
        <div id="radioselection">
            <span :key="item+index" v-for="(item, index) in signs">
                <input type="radio" :id="item" name="sign" :value="item" 
                    :checked="selectedSign == item"
                    v-model="selectedSign">
                <label :for="item">{{item}}</label>
                &nbsp;
            </span>
        </div>
        <div>
            <video @click="freeze()" id="video" width="320" height="240" autoplay></video>
            <canvas @click="addImage()" id="canvas" width="320" height="240"></canvas>
        </div>
        <div id="listOPics">
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
        name: 'Capture',
        data: function () {
            return {
                processing: false,
                message: '',
                video: null,
                canvas: null,
                signs: ['rock', 'paper', 'scissors', 'lizard', 'spock'],
                selectedSign: 'rock',
                list: []
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
            freeze: function () {
                this.canvas.drawImage(this.video, 0, 0, 320, 240);
            },
            addImage: function () {
                let c = document.getElementById('canvas')
                this.list.push({
                    type: this.selectedSign,
                    image: c.toDataURL()
                })
                window.scrollTo(0,document.querySelector("#app").scrollHeight)
            },
            removeImage: function (index) {
                this.list.splice(index, 1)
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
    video, canvas {
        border: solid 1px gray;
        cursor: pointer;
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
        top: 10px;
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