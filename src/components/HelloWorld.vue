<template>
  <div style="margin:0 auto;width:50%;">
    <h3 style="color:#20A0FF">VUE——时间滑块选择控件</h3>
    <hr>
    <div style="margin:0 auto;padding-top:40px;width:300px;height: 180px;">
      <vue-time-slider :width="width"  :times="times" @listenTimesChangeEvent="bindTimesChangeEvent"    ></vue-time-slider>
    </div>
    <hr>
    <div  >所选时间：{{this.times}}</div>

  </div>
</template>

<script>
  import vueTimeSlider from './vue-time-slider';
  export default {
    components: { vueTimeSlider },
    data () {
      return {
        times :[],
        width: 300,
      }
    },

    methods: {
      bindTimesChangeEvent :function(data){
        this.times = data;
        console.log(JSON.stringify(data));
      }
    },
    mounted(){ }
  }
</script>
