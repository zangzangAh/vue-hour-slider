<template>
  <div   @mouseup="handleMouseStop"  @mousedown="handleDivMouseStart"   @mousemove="handleDivMouseMove"  ondragstart="return false;" oncontextmenu="return false;"  >
    <table ref='time_slider_dev' id="time_slider_dev"   :style="divCursorStyle()"   >
      <tr v-for="rowIndex in deliverRows" ref='tr_time_slider_bar'  style="border:1px solid black;height:50px" >
        <td v-for="(time,index) in deliverTimes" ref='td_time_slider_bar' v-if="index>=(rowIndex-1)*numPerRow&&index<rowIndex*numPerRow"  >
          <div ref='div_time_text_start'   :id="'div_time_text_start_'+time.value"    class="time_slider_text"          :style="startTextStyle(index,time.value)"      v-if="index%numPerRow==0"><span   :style="startTextStyle(index,time.value)" >{{time.value-1}}</span></div>
          <div ref='div_time_slider_button':id="'div_time_slider_button_'+time.value" class="time_slider_button_start" :style="buttonStyle(index,time.value)"     @mousedown="handleMouseStart"  ></div>
          <div ref='div_time_slider_bar'   :id="'div_time_slider_bar_'+time.value"    class="time_slider_bar"           :style="barStyle(index,time.value)"        @click="handleTimeClick(index,time.value)" > </div>
          <div ref='div_time_text_stop'    :id="'div_time_text_stop_'+time.value"     class="time_slider_text"          :style="stopTextStyle(index,time.value)"      > <span  :style="stopTextStyle(index,time.value)"  >{{time.value}}</span></div>
          <div ref='div_time_slider_button' id="div_time_slider_button_25"          class="time_slider_button_start"  style="left:100%;top:79.16%;" v-if="time.value==24"  @mousedown="handleMouseStart"  ></div>
        </td>
      </tr>
    </table>
  </div>
</template>
<script  >
  export default {
    name: 'timeSlider',
    props: {
      times: {
        type: Array,
        default: []
      },
      width:{
        type:Number,
        default: 300
      }
    },
    data() {
      return {
        timesArray :this.times,
        deliverRows :[1,2,3],
        deliverTimes : [
          {'value':1, 'name':'00~01' },{'value':2, 'name':'01~02' },{'value':3, 'name':'02~03' },{'value':4, 'name':'03~04' },{'value':5, 'name':'04~05' },{'value':6, 'name':'05~06' },{'value':7, 'name':'06~07' },{'value':8, 'name':'07~08' },
          {'value':9, 'name':'08~09' },{'value':10,'name':'09~10' },{'value':11,'name':'10~11' },{'value':12,'name':'11~12' },{'value':13,'name':'12~13' },{'value':14,'name':'13~14' },{'value':15,'name':'14~15' },{'value':16,'name':'15~16' },
          {'value':17,'name':'16~17' },{'value':18,'name':'17~18' },{'value':19,'name':'18~19' },{'value':20,'name':'19~20' },{'value':21,'name':'20~21' },{'value':22,'name':'21~22' },{'value':23,'name':'22~23' },{'value':24,'name':'23~24' }
        ],
        numPerRow:8,
        lengthPerStep:12.5,
        disabledTextColor:'black',
        disabledColor:'rgb(228, 231, 237)',
        disabledVisible:'hidden',
        enabledColor:'rgb(32, 160, 255)',
        enabledVisible:'visible',
        divCursor:'default' ,
        canDrag: false,
        dragStartIndex:-1,
        dragCurrentIndex:-1,
        dragStartRowIndex:-1,
        dragCurrentRowIndex:-1,
        startX:0,
        startY:0,
        currentX:0,
        currentY:0,
      };
    },

    watch: {
      times:{
        handler: function (val,oldVal) {
          this.times = val ;
          this.timesArray = this.times;
        },
        deep: true
      },
      timesArray:{
        handler: function (val,oldVal) {
          if(val!=oldVal){
            this.$emit("listenTimesChangeEvent",this.timesArray);
          }
        },
        deep: true
      },
    },

    methods: {
      //初始化参数
      clear:function(){
        this.startX=0 ;
        this.startY=0 ;
        this.currentX = 0 ;
        this.currentY = 0 ;
        this.canDrag = false;
        this.dragStartIndex =-1;
        this.dragCurrentIndex =-1;
        this.dragStartRowIndex=-1;
        this.dragCurrentRowIndex=-1;
        this.divCursor = 'default' ;
      } ,
      getButtonVisibility:function(time)  {
        let visibility = this.disabledVisible  ;
        if(time == 1 ) visibility = this.enabledVisible ;
        else  if(this.timesArray.indexOf(time)>-1){
          if(this.timesArray.indexOf(time-1)>-1)  visibility = this.disabledVisible ;
          else visibility = this.enabledVisible;
        }else {
          if(this.timesArray.indexOf(time-1)>-1)  visibility = this.enabledVisible ;
          else visibility = this.disabledVisible;
        }
        return visibility;
      },

      getTextColor:function(time)   { return this.timesArray.indexOf(time) > -1 ? this.enabledColor : this.disabledTextColor; },
      getTextTop:function(index)    { return (1/3*100)*(parseInt(index/this.numPerRow))  ; },
      getTextStop:function(index)   { return (index%this.numPerRow +1) * this.lengthPerStep; },
      getBarColor:function(time)    { return this.timesArray.indexOf(time) > -1 ? this.enabledColor : this.disabledColor;  },
      getButtonTop:function(index)  { return (1/3*100)*(parseInt(index/this.numPerRow))+this.lengthPerStep; },
      getButtonLeft:function(index) { return (index%this.numPerRow   ) *  this.lengthPerStep; },
      //设置文本样式
      startTextStyle:function(index,time){
        let left  = this.getButtonLeft (index);
        let top   = this.getTextTop(index);
        let color = this.getTextColor(time);
        return `left:${left}%;top:${top}%;color:${color}`;
      },
      //设置文本显示
      stopTextStyle:function(index,time){
        let left  = this.getTextStop (index);
        let top   = this.getTextTop(index);
        let color = this.getTextColor(time);
        return `left:${left}%;top:${top}%;color:${color}`;
      },
      barStyle:function(index,time){
        let color = this.getBarColor(time);
        let cursor = 'pointer' ;
        if(this.canDrag) cursor = 'grab' ;
        return `backgroundColor:${color};cursor:${color};`;
      },

      divCursorStyle:function(){
        return `cursor:${this.divCursor};width:${this.width}px;height: 150px;position: absolute;`;
      },
      //设置开始按钮显示
      buttonStyle:function(index,time){
        let left= this.getButtonLeft (index);
        let top = this.getButtonTop(index);
        let visibility = this.getButtonVisibility(time)  ;
        return `left:${left}%;top:${top}%;visibility:${visibility}`;
      },

      //时间轴按钮点击时间
      handleTimeClick(index,time){
        this.setTimesArray();
        var addFlag = true ;
        for(var i = 0 ;i<this.timesArray.length ;i++){
          if(this.timesArray[i]==time){
            addFlag = false ;
            this.timesArray.splice(i,1);
            break;
          }
        }
        if(addFlag)  this.timesArray.push(time);
      },
      //鼠标左键点击开始事件
      handleMouseStart:function(event){
        this.clear() ;
        this.canDrag=true;//鼠标左键点击
        //记录鼠标指针位置
        this.startX = event.clientX;
        this.startY = event.clientY;
        this.divCursor = 'grab' ;
        this.dragStartIndex = Number(event.srcElement.id.split("_")[4])-1;
        this.addTimeButtonEventListener(this.dragStartIndex) ;
      },
      //鼠标左键点击结束事件
      handleMouseStop:function(event){
        this.clear();
        this.setTimesArray();
      },
      handleDivMouseStart:function(event){
        // this.canDrag =false ;
      },
      //添加鼠标监听事件
      addTimeButtonEventListener:function(index){
        this.$refs.div_time_slider_button[index].addEventListener('mousemove', this.handleMouseMove);
        this.$refs.div_time_slider_button[index].addEventListener('mouseup', this.handleMouseStop);
        this.$refs.div_time_slider_button[index].addEventListener('contextmenu', this.handleMouseStop);
      },
      //移除鼠标监听事件
      removeTimeButtonEventListener:function(index){
        // this.$refs.div_time_slider_button[index].removeEventListener('mousemove', this.handleMouseMove);
        this.$refs.div_time_slider_button[index].removeEventListener('mouseup', this.handleMouseStop);
        this.$refs.div_time_slider_button[index].removeEventListener('contextmenu', this.handleMouseStop);

      },
      //根据滑块选择赋值
      setTimesArray:function() {
        this.canDrag = false;
        this.timesArray = [];
        for(var i = 0; i < 24 ; i++) {
          if (this.$refs.div_time_slider_bar[i].style.backgroundColor != this.disabledColor)
            this.timesArray.push(i + 1);
        }
      },
      //滑块移动方法
      handleDivMouseMove:function(event){
        if(this.canDrag){
          // 计算滑动步数
          this.currentX = event.clientX; //X坐标
          this.currentY = event.clientY;//Y坐标

          this.dragStartRowIndex = this.dragStartIndex==24 ? 2 : Math.floor((this.dragStartIndex)/this.numPerRow);// 计算开始行数索引
          this.dragCurrentRowIndex =  this.dragStartRowIndex +  Math.round(( this.currentY - this.startY ) / 50);

          var  steps = Math.round(((this.currentX - this.startX) / this.sliderDevWidth * 100) / this.lengthPerStep)   ;//滑动步数

          this.dragCurrentIndex = this.dragStartIndex +  steps;//计算滑动当前索引
          if(this.dragCurrentRowIndex != this.dragStartRowIndex){
            this.dragCurrentIndex = this.dragStartIndex +  steps + 8*(this.dragCurrentRowIndex-this.dragStartRowIndex);//计算滑动当前索引
          }
          //滑动条件——1.开始索引不等于滑动索引 2.开始索引与滑动索引仅支持1个步数 3.开始行与当前行不支持跨行
          if(this.dragCurrentIndex != this.dragStartIndex && Math.abs(this.dragCurrentIndex-this.dragStartIndex )==1
            && (this.dragCurrentRowIndex==this.dragStartRowIndex||Math.abs(this.dragCurrentRowIndex-this.dragStartRowIndex) ==1 )) {
            if(this.dragCurrentIndex>this.dragStartIndex) this.dragMoveRight();
            else if(this.dragCurrentIndex<this.dragStartIndex) this.dragMoveLeft();
          }
        }
      },
      //向右滑动
      dragMoveRight:function(){
        let  barBgColor = this.$refs.div_time_slider_bar[this.dragStartIndex].style.backgroundColor!=this.enabledColor? this.enabledColor : this.disabledColor ;
        let  textColor  = this.$refs.div_time_slider_bar[this.dragStartIndex].style.backgroundColor!=this.enabledColor? this.enabledColor : this.disabledTextColor    ;

        this.setBarBgColor(this.dragStartIndex,barBgColor,textColor) //设置时间条颜色
        if(this.dragStartIndex != 0)  this.setButtonStyle(this.dragStartIndex,this.disabledVisible);//隐藏开始按钮

        let currentIndex = this.dragCurrentIndex ; //记录当前索引
        //滑动后合并后来相同设置的时间框
        for(let index = this.dragCurrentIndex+1;index < (this.dragStartRowIndex+1) * this.numPerRow+1;index++){
          let nextColor = this.$refs.div_time_slider_bar[index-1].style.backgroundColor ;
          if( nextColor == barBgColor ) this.dragCurrentIndex = index  ;
          else break;
        }
        //隐藏合并中出现的按钮
        if (currentIndex != this.dragCurrentIndex){
          this.canDrag =false ;
          for(let index = currentIndex ; index < this.dragCurrentIndex ;index ++){
            this.setButtonStyle(index,this.disabledVisible);
          }
        }
        //设置当前按钮显示
        if(this.dragCurrentIndex %this.numPerRow==0 && this.dragCurrentIndex!=24 && this.$refs.div_time_slider_bar[this.dragCurrentIndex].style.backgroundColor==barBgColor) {
          this.setButtonStyle(this.dragCurrentIndex,this.disabledVisible);
        }else this.setButtonStyle(this.dragCurrentIndex,this.enabledVisible);
        //更新开始位置
        this.updateStartIndex();
      },
      //向左滑动
      dragMoveLeft:function(){
        let  barBgColor =  this.$refs.div_time_slider_bar[this.dragStartIndex-1].style.backgroundColor==this.enabledColor? this.disabledColor : this.enabledColor  ;
        let  textColor = this.$refs.div_time_slider_bar[this.dragStartIndex-1].style.backgroundColor==this.enabledColor? this.disabledTextColor : this.enabledColor  ;

        this.setBarBgColor(this.dragCurrentIndex,barBgColor,textColor) //设置时间条颜色
        if(this.dragStartIndex != 24)  this.setButtonStyle(this.dragStartIndex,this.disabledVisible);//隐藏开始按钮

        let currentIndex = this.dragCurrentIndex ;//记录当前索引

        //滑动后合并后来相同设置的时间框
        for(let index = this.dragCurrentIndex-1;index >  this.dragStartRowIndex * this.numPerRow-1;index--){
          let nextColor = this.$refs.div_time_slider_bar[index].style.backgroundColor ;
          if( nextColor == barBgColor ) this.dragCurrentIndex -- ;
          else break;
        }
        //隐藏合并中出现的按钮
        if (currentIndex != this.dragCurrentIndex){
          this.canDrag =false ;
          for(let index = currentIndex ; index > this.dragCurrentIndex ;index--){
            this.setButtonStyle(index,this.disabledVisible);
          }
        }
        //设置当前按钮显示
        if(this.dragCurrentIndex %this.numPerRow==0 && this.dragCurrentIndex !=0 && this.$refs.div_time_slider_bar[this.dragCurrentIndex-1].style.backgroundColor==barBgColor) {
          this.setButtonStyle(this.dragCurrentIndex,this.disabledVisible);
        }else this.setButtonStyle(this.dragCurrentIndex,this.enabledVisible);
        //更新开始位置
        this.updateStartIndex();

      },

      //设置开始按钮样式
      setButtonStyle:function(index,visibility){
        if(index>=0&&index<24){
          let left = this.getButtonLeft(index);
          let top =  this.getButtonTop(index);
          if(index==0) visibility = this.enabledVisible ;
          this.$refs.div_time_slider_button[index].style = `left:${left}%;top:${top}%;visibility:${visibility}`;
        }
      },

      //设置时间框样式
      setBarBgColor:function(index ,bgColor,textColor){
        if(index >= 0 && index < 24 ){
          this.$refs.div_time_slider_bar[index].style.backgroundColor =bgColor;
          this.$refs.div_time_text_stop[index].style.color = textColor;
          if(index%this.numPerRow==0){
            let textStartIndex = parseInt(index/this.numPerRow) ;
            this.$refs.div_time_text_start[ textStartIndex].style.color = textColor;
          }
        }
      },



      updateStartIndex:function(){
        this.dragStartIndex = this.dragCurrentIndex ;
        if(this.dragStartIndex == 24){
          this.startX = this.lastSliderButtonLeft ;
          this.startY = this.lastSliderButtonTop ;
        }else{
          this.startX = this.$refs.div_time_slider_bar[this.dragStartIndex].getBoundingClientRect().left ;
          this.startY = this.$refs.div_time_slider_bar[this.dragStartIndex].getBoundingClientRect().top ;
        }
        // if(this.dragCurrentIndex % this.numPerRow==0  ) {
        //   this.canDrag =false ;
        // }
      },
    },

    computed: {
      sliderDevWidth() {  return this.$refs.time_slider_dev.clientWidth ;  },
      lastSliderButtonLeft() {  return this.$refs.div_time_slider_button[24].getBoundingClientRect().left;  },
      lastSliderButtonTop()  {  return this.$refs.div_time_slider_button[24].getBoundingClientRect().top;  },
    },
    mounted() {  },
  };
</script>
<style   >
  .time_slider_text  {
    width:12.5%;
    height: 5px;
    line-height: 5px;
    position: absolute;
    -moz-user-select: none;
    -khtml-user-select: none;
    user-select: none;
  }
  .time_slider_bar  {
    width:10%;
    height: 5px;
    margin-left:2.8%;
    background-color: #e4e7ed;
    border-radius: 3px;
    position: absolute;
    cursor:pointer;
  }
  .time_slider_button_start {
    position:absolute;
    width: 12px;
    height: 12px;
    border: 2px solid #409eff;
    background-color: #fff;
    border-radius: 50%;
    cursor:grab;
    z-index:100
  }
  .time_slider_button_stop {
    position:absolute;
    width: 12px;
    height: 12px;
    border: 2px solid #409eff;
    background-color: #fff;
    border-radius: 50%;
    cursor:grab;
    z-index:100
  }
</style>
