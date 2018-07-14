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
        width: 400px;
        height: 30px;
        margin-top: 10px;
        margin-bottom: 10px;
        margin-left: auto;
        margin-right: auto;
    }

</style>
