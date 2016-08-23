<template>

    <div class="app">
        <div class="left">

            <div class="list" v-if="!status && player.status == 3">
                <h2><i class="fa fa-road" aria-hidden="true"></i> Geçmiş Yolculuklar</h2>

                <ul>
                    <li v-for="l in list" @click="replay(l._id)">
                        <div class="start">{{l.start}}</div><div class="divider"><i class="fa fa-exchange" aria-hidden="true"></i></div><div class="end">{{l.end}}</div>
                    </li>
                </ul>

            </div>

            <div id="map" :class="{blur: (player.status == 3 && !status)}">
                <maps :center.sync="maps.center" :zoom.sync="maps.zoom" :heading.sync="data[0][2]">
                    <marker
                            :position.sync="maps.center"
                    ></marker>
                </maps>
            </div>
            <div v-if="player.status != 3 && !status">
                <slider v-if="player.size" :max.sync="player.size" :value.sync="player.frame" ></slider>
                <div class="control">
                    <i @click="play()" v-if="player.status == 2" class="fa fa-play" aria-hidden="true"></i>
                    <i @click="pause()" v-if="player.status == 1" class="fa fa-pause" aria-hidden="true"></i>
                    <i @click="stop()" class="fa fa-stop" aria-hidden="true"></i>
                    {{time(player.timeFrame)}} / {{time(player.size)}}

                    <span class="pull-right" @click="speed()"><i class="fa fa-fast-forward" aria-hidden="true"></i><sup>{{player.speed}}</sup></span>
                </div>
            </div>

        </div>

        <div class="right">

            <div class="boxes">
                <div class="box">
                    <gauge :value.sync="data[1][2]" max="210" title="Araç Hızı km/h"></gauge>
                </div>
                <div class="box">
                    <gauge :value.sync="data[1][1]" max="5000" title="Motor Devri (rpm)"></gauge>
                </div>
                <div class="box">
                    <h2>
                        {{data[1][3]}}°
                        <small>Radyatör Sıcaklığı</small>
                    </h2>
                </div>
                <div class="box">
                    <h2>
                        {{data[1][0]}}%
                        <small>Motor Yükü</small>
                    </h2>
                </div>
            </div>

            <a href="https://github.com/MetinSeylan/Kaput" target="_blank"><i class="fa fa-github" aria-hidden="true"></i> Kaynak Kodları</a>
        </div>

    </div>



</template>


<script type="text/babel">
    import {load, loaded, Map, Marker} from 'vue-google-maps';
    load(null,'3.24');

    import slider from './partials/slider.vue';
    import gauge from './partials/gauge.vue';


    export default{
        components:{
            slider: slider,
            maps: Map,
            marker: Marker,
            gauge: gauge
        },
        data(){
            return {
                player: {
                    status: 3,
                    speed: 1,
                    play: false,
                    size: 0,
                    frame: 0,
                    timeFrame: 0
                },
                maps: {

                    center: {"lat": 41.037502, "lng": 28.987561},
                    zoom: 16,
                    lat: 0,
                    lng: 0
                },
                list: [],
                status: false,
                data: [
                    [
                        -33.315629, //longitude
                        47.785156, //latitude
                        30 //heading
                    ],
                    [
                        0, //load
                        0, //rpm
                        0, //speed
                        0 //coolant
                    ]
                ]
            }
        },
        events: {
            pause(){
                this.pause();
            },
            play(){
                this.play();
            },
            setFrame(id){
                this.frame(id);
            }
        },
        methods: {
            replay(id){
                this.$socket.emit('replay', id);
            },
            frame(id){
                this.$socket.emit('change_frame', id);
            },
            pause(){
                this.$socket.emit('player_pause');
            },
            play(){
                this.$socket.emit('player_play');
            },
            stop(){
                this.$socket.emit('player_stop');
                this.player.size = 0;
                this.player.frame = 0;
            },
            speed(){

                let speed = this.player.speed+1;

                if(speed == 5) speed = 1;

                this.player.speed = speed;
                this.$socket.emit('player_speed', speed);

            },
            time(size){
                let min = Math.floor((size / 4).toFixed(0) / 60);
                let sec = String(((size / 4).toFixed(0) - (min * 60)));

                let pad = String("00").substring(0, 2 - sec.length) + sec;

                return min + ':' + pad;
            }
        },
        sockets: {
            data(data){
                this.data = data.data;

                if (data.frame == 0) this.status = true;

                if(this.player.status == 1) this.player.frame = data.frame;

                this.player.timeFrame = data.frame;


                this.maps.center.lat = data.data[0][1];
                this.maps.center.lng = data.data[0][0];


                this.$broadcast('g-panTo', new window.google.maps.LatLng(data.data[0][1], data.data[0][0]));

            },
            status(status){
                this.status = status;
            },
            list(list){
                this.list = list;
            },
            playerStatus(status){
                this.player.status = status;
            },
            playerInfo(data){
                this.player.size = data.size;
            }
        }
    }
</script>

<style type="style/sass" lang="sass">
    @import "style"
</style>