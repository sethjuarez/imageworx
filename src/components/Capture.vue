<template>
    <div>
        <div>
            <video v-on:click="freeze()" id="video" width="320" height="240" autoplay></video>
            <canvas v-on:click="addImage()" id="canvas" width="320" height="240"></canvas>
        </div>
        <div>
            <span :key="item+index" v-for="(item, index) in signs">
                <input type="radio" :id="item" name="sign" :value="item" 
                    :checked="selectedSign == item"
                    v-model="selectedSign">
                <label :for="item">{{item}}</label>
                &nbsp;
            </span>

        </div>
        <div id="listOPics">
            <ul class="imagelist" :key="index" v-for="(item, index) in list">
                <li class="imgitem">
                    <div>{{item.type}}</div>
                    <img height="120" width="160" :src="item.image" />
                </li>
            </ul>
            <div class="btn">
                <button type="button" v-if="list.length > 0" v-on:click="submitImages()">Submit Training Data</button>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'Capture',
        data: function () {
            return {
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
            },
            submitImages: function () {
                alert('made it!')
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
        padding-top: 50px;
    }

    .imagelist {
        list-style-type: none;
        padding: 0px;
    }

    .btn {
        text-align: center;
    }
</style>