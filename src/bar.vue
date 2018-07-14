<template>
    <div class="bar" v-bind:style="{ 'background-color': activeColor, height:hh}"></div>
</template>

<script>
    export default {
        name: 'bar',
        props: {
            height: Number,

        },
        data() {
            return {
                activeColor: "rgba(255, 255, 255, 0.5)",
            }

        }, methods: {
            ChangeToWhite() {
                this.activeColor = 'rgba(255, 255, 255, 1.0)';
            },
            ChangeToHalf() {
                this.activeColor = 'rgba(255, 255, 255, 0.5)';
            },
            ChangeToRed() {
                this.activeColor = '#FF4949';
            }, change() {
                if (this.activeColor === "rgba(255, 255, 255, 0.5)") {
                    this.ChangeToWhite();
                } else {
                    this.ChangeToHalf();
                }
            }
        },
    }
</script>

<style>
    .bar {
        width: 300px;
        height: 15px;
        margin-top: 10px;
        margin-bottom: 10px;
        margin-left: auto;
        margin-right: auto;
    }

</style>

<!--
<template>
    <div class="bar-wrap">
        <div class="bar" :style="{'height': (height + 1) + 'px'}"></div>
    </div>
</template>

<script>
    export default {
        name: 'bar',
        props: {
            height: Number,

        },
        data() {
            return {

            }
        },
    }
</script>

<style>
    .bar-wrap {
        position: absolute;
        width: 5px;
        margin: 2px;
        display: inline-block;
    }
    .bar {
        width: 5px;
        background: white;
        position: absolute;
        margin: auto;
        top: 0; left: 0; bottom: 0; right: 0;
        border-radius: 1px;
    }
</style>
-->
