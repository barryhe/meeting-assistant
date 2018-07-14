<template>
            <div id="app">

              <div id="background">

                <div id="container">
                  <div id="bars_container">
                    <bar v-for="i in num" ref="bar"></bar>
                  </div>

                  <div id="basic_setup">
                    <h3>Basic Setup</h3>
                    <hr>

                    <div class="item">
                      <template>
                        <span>Threshold: </span> <el-input-number v-model="threshold" :step="10" size="large"></el-input-number>
                      </template>
                    </div>
                    <div class="item">
                      <span>Beeper: </span>
                      <el-switch v-model="beeper_switch"
                              on-color="#13ce66"
                              off-color="#ff4949"

                      >

            </el-switch>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
    import Vue from "vue";
    import Bar from "./bar.vue";
    import element from 'element-ui';
    import volumeMeter from 'volume-meter';
    import getusermedia from 'getusermedia';

    export default {
        components: {Bar, element, volumeMeter},
        name: 'app',
        data () {
            return {
                num: 20,      // number of bars
                factor: 5,
                threshold: 50,
                prev: 0,
                toWait: 50, // milliseconds

                beeper_switch: false,
            };
        },
        created: function () {
            this.setup();
        },
        methods: {
            pathGenerator () {
                return "/dist/music/" + this.song + ".mp3";
            },
            setup: function() {
                this.prev = (new Date()).getTime();

                var self = this;
                if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                    console.log('getUserMedia supported.');

                    getusermedia({ audio: true, video: false }, function (err, stream) {
                        if (err) {
                            return console.error("Error: " + err);
                        }

                        var ctx = new AudioContext();
                        var meter = volumeMeter(ctx, self.updateBars);

                        var src = ctx.createMediaStreamSource(stream);
                        src.connect(meter);
                        src.connect(ctx.createGain());

                        stream.onended = function () {
                            meter.stop()
                        }
                    });
                } else {
                    console.log('getUserMedia not supported on your browser!');
                }
            },
            updateBars(volume) {
                let time = (new Date()).getTime();

                if (time - this.prev >= this.toWait) {
                    this.prev = time;
                    // update
                }  else {
                    // wait;
                    return;
                }

                if (volume < 0.5) {
                    volume = 0;
                }
                let isRed = volume > this.threshold;
                for (let i = this.num - 1; i >= 0; --i) {
                    // update the color of bars

                    if (volume > 0) {
                        if (isRed) {
                            this.$refs.bar[i].ChangeToRed();
                        } else {
                            this.$refs.bar[i].ChangeToWhite();
                        }
                        volume -= this.factor;
                    } else {
                        this.$refs.bar[i].ChangeToHalf();
                    }
                }
            },
            reverseData() {
                let x = this.mData.slice().reverse();
                return x;
            },
            playAnother () {

            },
            change() {
                for (let b = 0; b < this.num; ++b) {
                    // this.refs.bar guaranteed to be an array
                    this.$refs.bar[b].change();
                }
            }
        },
        mounted() {
        }
    }
    // linear-gradient(to bottom, rgba(0,168,255,0.5), #157cad)
</script>

<style>
  
  body {
    padding: 0;
    margin: 0;

    font-family: "Arial", sans-serif;
  }
  
  button {
    position: absolute;
  }

  #background {
    background: linear-gradient(to top, rgba(0,168,255,1), #42b983);
    position: absolute;
    width: 100vw;
  }

  #container {
    width: 750px;
    margin-left: auto;
    margin-right: auto;
    margin-top: 50px;
    margin-bottom: 100px;
    word-break: break-all;

    color: white;
  }

  #bars_container {
    width: 100%;
    text-align: center;
  }

  #basic_setup {
    margin: 30px 0;
  }

  #basic_setup h3 {
    font-size: 60px;
    text-align: center;
    margin-bottom: 10px;
  }

  hr {
    margin: 10px;
    border: 0;
    background-color: #eee;
    height: 1px;
  }

  .item > span {
    font-size: 30px;
    vertical-align: middle;
  }
  .item {
    margin: 30px auto;
    width: 50%;
  }


</style>
