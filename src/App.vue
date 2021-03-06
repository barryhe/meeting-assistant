<template>
    <div id="app">

      <div id="background" class="disable-doubletap-to-zoom">

        <div id="container">
          <div id="bars_container">
            <bar v-for="i in num" ref="bar"></bar>
          </div>

          <div id="basic_setup">
            <h3 class="show_text">Basic Setup</h3>
            <hr>

            <div class="item" v-on:click="checkThreshold();">
                <span class="show_text">Threshold: </span>
                  <template>
                      <el-input-number v-model="threshold" :step="factor"></el-input-number>
                  </template>
            </div>
            <div class="item">
                <span class="show_text">Warning: </span>
                <label class="switch">
                    <input type="checkbox" checked v-on:click="disableWarning();">
                    <span class="slider round"></span>
                </label>
            </div>

            <div class="item">
              <span class="show_text">Trigger Mode: </span>
              <label class="switch">
                  <input type="checkbox" v-on:click="switchMode();">
                  <span class="slider round"></span>
              </label>
            </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
    import Vue from "vue";
    import Bar from "./bar.vue";
    import volumeMeter from 'volume-meter';
    import getusermedia from 'getusermedia';
    import element from 'element-ui';
    import NoSleep from 'nosleep.js';

    export default {
        components: {Bar, element, volumeMeter, NoSleep},
        name: 'app',
        data () {
            return {
                num: 20,      // number of bars
                factor: 5,
                threshold: 35,
                prev: 0,
                toWait: 50, // milliseconds
                min: 0,
                max: 100,
                beeper: null,
                beeperGain: null,

                /* trigger mode setup */
                startMadness: false,
                madnessPrev: 0,
                madnessMaxDuration: 300, // ms

                startPeace: false,
                peacePrev: 0,
                peaceMaxDuration: 1500, // ms

                isTriggerMode: false,
                /* end of trigger mode */

                // enable/disable beeper
                beeper_switch: true,

                // switch to polynomial mode
                smoothing_factor: 1000.0,

                // no sleep
                noSleep: new NoSleep(),
            };
        },
        created: function () {
            this.setup();
        },
        methods: {
            pathGenerator () {
                return "/dist/beep.mp3";
            },
            setup: function() {
                this.prev = (new Date()).getTime();

                /* start mic setup */
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
                            meter.stop();
                        }
                    });
                } else {
                    console.log('getUserMedia not supported on your browser!');
                }
                /* end of recording */

                /* disable zoom by double click */
                var lastTouch = 0;
                function preventZoom(e) {
                    var t2 = e.timeStamp;
                    var t1 = lastTouch || t2;
                    var dt = t2 - t1;
                    var fingers = e.touches.length;
                    lastTouch = t2;

                    if (!dt || dt >= 300 || fingers > 1) {
                        return;
                    }
                    resetPreventZoom();
                    e.preventDefault();
                    e.target.click();
                }
                function resetPreventZoom() {
                    lastTouch = 0;
                }

                document.addEventListener('touchstart', preventZoom, false);
                document.addEventListener('touchmove', resetPreventZoom, false);
                /* end of double click */

                /* start of beeper setup */
                window.AudioContext = window.AudioContext || window.webkitAudioContext;
                var context = new AudioContext();
                // var analyser = context.createAnalyser();
                var source;
                self.beeperGain = context.createGain();
                this.beeper = new Audio(this.pathGenerator());
                this.beeper.controls = true;
                this.beeper.loop = true;

                source = context.createMediaElementSource(this.beeper);
                source.connect(self.beeperGain);
                self.beeperGain.connect(context.destination);
                /* end of beeper */

                /* prevent auto sleeping */

                console.log("Enabled noSleep");
                this.noSleep.enable();
                /* end of sleeping prevention */
            },
            updateBars(volume) {
                let time = (new Date()).getTime();

                // beeper triggering
                if (this.isTriggerMode) {
                    if (volume > this.threshold) {
                        if (!this.startMadness) {
                            // exceed the maximum time
                            this.startMadness = true;
                            this.madnessPrev = time;
                        } else if (time - this.madnessPrev > this.madnessMaxDuration) {
                            this.playBeeper();
                        }
                    } else {
                        // two solutions:
                        // 1. beep after 0.3s of madness
                        // 2. beep if volume > threshold

                        if (!this.startPeace) {
                            this.startPeace = true;
                            this.peacePrev = time;
                        } else {
                            if (time - this.peacePrev > this.peaceMaxDuration) {
                                this.startPeace = false;
                                this.startMadness = false;
                                this.stopBeeper();
                            }
                        }
                    }
                } else {
                    // smooth mode
                    if (volume > this.threshold) {
                        let diff = volume - this.threshold;
                        let newVal = Math.pow(diff, 1.3) / this.smoothing_factor;
                        this.beeperGain.gain.value = newVal;
                        // this.beeperGain.gain.setTargetAtTime(0.5, this.beeper.currentTime, 0);
                        this.playBeeper();
                    } else {
                        this.stopBeeper();
                    }
                }

                // volume meter rendering
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
            playBeeper() {
                if (this.beeper_switch) {
                    this.beeper.play();
                }
            },
            stopBeeper() {
                if (this.beeper !== null) {
                    this.beeper.pause();
                    this.beeper.currentTime = 0;
                }
            },
            disableWarning() {
                this.beeper_switch = !this.beeper_switch;
            },
            change() {
                for (let b = 0; b < this.num; ++b) {
                    // this.refs.bar guaranteed to be an array
                    this.$refs.bar[b].change();
                }
            }, checkThreshold() {
                if (this.threshold < this.min) {
                    this.threshold = this.min;
                } else if (this.threshold > this.max) {
                    this.threshold = this.max;
                }
            }, switchMode() {
                this.isTriggerMode = !this.isTriggerMode;
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

  .show_text {
      user-select: none;
      -moz-user-select: none;
      -khtml-user-select: none;
      -webkit-user-select: none;
      -o-user-select: none;
  }

  #background {
        background: linear-gradient(to top, rgba(0,168,255,1), #42b983);
        position: absolute;
        width: 100vw;
        -webkit-tap-highlight-color: rgba(0,0,0,0);
        -webkit-tap-highlight-color: transparent;
    }

  #container {
    width: 850px;
    margin-left: auto;
    margin-right: auto;
    margin-top: 150px;
    margin-bottom: 400px;
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
    font-size: 100px;
    text-align: center;
    margin-bottom: 10px;
  }

  hr {
    margin: 25px;
    border: 0;
    background-color: #eee;
    height: 1px;
  }

  .item > span {
      font-size: 60px;
  }

  .item {
    margin: 30px auto;
    width: 100%;
  }



  /* sliders */
  .switch {
      position: relative;
      display: inline-block;
      width: 240px;
      height: 68px;
      top: 12px;
  }

  .switch input {display:none;}

  .slider {
      position: absolute;
      cursor: pointer;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background-color: #ff4949;
      -webkit-transition: .4s;
      transition: .4s;
  }

  .slider:before {
      position: absolute;
      content: "";
      height: 52px;
      width: 52px;
      left: 8px;
      bottom: 8px;
      background-color: white;
      -webkit-transition: .2s;
      transition: .2s;
  }

  input:checked + .slider {
      background-color: #13ce66;
  }


  input:checked + .slider:before {
      -webkit-transform: translateX(170px);
      -ms-transform: translateX(170px);
      transform: translateX(170px);
  }

  /* Rounded sliders */
  .slider.round {
      border-radius: 34px;
  }

  .slider.round:before {
      border-radius: 50%;
  }

  .disable-doubletap-to-zoom {
      touch-action: none;
  }

</style>
