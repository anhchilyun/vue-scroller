<template>
  <div class="scroll-box" @mouseenter="onMouseEnter" @mouseleave="onMouseLeave">
    <div class="scroll__wrap">
      <slot></slot>
    </div>
    <div class="scroll__bar" type="y" v-show="yBar.show">
      <div class="scroll__slider"
           ref="ySlider"
           :class="{active: yMouseDown}"
           :style="{height: yBar.size + '%', top: yBar.sliderPos + '%'}"
           @mousedown="onMouseDown($event, 'y')"
      ></div>
    </div>
    <div class="scroll__bar" type="x" v-show="xBar.show" >
      <div class="scroll__slider"
           ref="xSlider"
           :class="{active: xMouseDown}"
           :style="{width: xBar.size + '%', left: xBar.sliderPos + '%'}"
           @mousedown="onMouseDown($event, 'x')"
      ></div>
    </div>
  </div>
</template>
<script>
  export default {
    name: 'goofyScroll',
    data(){
      return {
        yMouseDown: false,
        xMouseDown: false,
        barType: null,
        yBar: {
          show: false,
          sliderPos: 0,
          start: 0,
          offset: 0,
          size: 0,
          pxSize: 0
        },
        xBar: {
          show: false,
          sliderPos: 0,
          start: 0,
          offset: 0,
          size: 0,
          pxSize: 0
        }
      }
    },
    mounted(){
      this.wrapEl = this.$el.querySelector('.scroll__wrap')
      document.addEventListener('mousemove', this.onMouseMove)
      document.addEventListener('mouseup', this.onMouseUp)
      this.wrapEl.addEventListener('scroll', this.onScroll)
      window.onresize = () => {
        this.update()
      }
      this.update()

    },
    methods: {
      update(){
        let wrapH  = this.wrapEl.clientHeight
        let viewH = this.wrapEl.scrollHeight
        this.yBar.show = !(viewH <= wrapH)
        this.$emit('yBar', this.yBar.show)
        if(this.yBar.show){
          this.yBar.size = Math.round(wrapH / viewH * 100)
        }
        // x轴滚动条
        let wrapW = this.wrapEl.clientWidth
        let viewW = this.wrapEl.scrollWidth
        this.xBar.show = !(viewW <= wrapW)
        this.$emit('xBar', this.xBar.show)
        if(this.xBar.show){
          this.xBar.size = Math.round(wrapW / viewW * 100)
        }
      },
      onMouseLeave(){
        this.update()
      },
      onMouseEnter(){
        this.update()
      },
      onMouseDown(e, type){
        let prefix = this.barType = type === "y" ? type : (type === "x"? type: null)
        if(prefix){
          this[prefix + 'MouseDown'] = true
          this[prefix + 'Bar'].start = e['page' + prefix.toUpperCase()]
          let sliderEl
          if(prefix === 'y'){
            sliderEl = this.$refs.ySlider
            this.yBar.pxSize = sliderEl.clientHeight
            this.yBar.offset = (sliderEl.offsetTop / sliderEl.clientHeight) * this.yBar.size
          }
          if(prefix === 'x'){
            sliderEl = this.$refs.xSlider
            this.xBar.pxSize = sliderEl.clientWidth
            this.xBar.offset = (sliderEl.offsetLeft / sliderEl.clientWidth) * this.xBar.size
          }
        }
      },
      onMouseMove(e){
        let isDown = this.barType && this[this.barType + 'MouseDown']
        if(isDown){
          let prefix = this.barType
          let bar = this[prefix + 'Bar']
          let move = e['page' + prefix.toUpperCase()] - bar.start
          let movePercent = Math.round((move / bar.pxSize * bar.size))
          let newPercent = bar.sliderPos = Math.min(100 - bar.size, Math.max(0, (movePercent + bar.offset)))
          if(prefix === 'y'){
            this.wrapEl.scrollTop = newPercent * this.wrapEl.scrollHeight / 100
          }
          if(prefix === 'x'){
            this.wrapEl.scrollLeft = newPercent * this.wrapEl.scrollWidth / 100
          }
        }
      },
      onMouseUp(){
        if(this.barType){
          this[this.barType + 'MouseDown'] = false
          this.barType = null
        }
      },
      onScroll(){
        if(!this.yMouseDown && !this.xMouseDown){
          this.yBar.sliderPos = Math.round(this.wrapEl.scrollTop / this.wrapEl.scrollHeight * 100)
          this.xBar.sliderPos = Math.round(this.wrapEl.scrollLeft / this.wrapEl.scrollWidth * 100)
        }
      }
    }
  }
</script>
<style lang="less" scoped>
  .scroll-box{
    width: 100%;
    height: 100%;
    padding-right: 15px;
    box-sizing: border-box;
    position: relative;
    .scroll__wrap{
      position: absolute;
      display: block;
      box-sizing: border-box;
      left: 0;
      right: 0;
      top:0;
      bottom: 0;
      overflow: scroll;
      margin-bottom: -17px;
      margin-right: -17px;
    }
    .scroll__bar{
      position: absolute;
      background: #eee;
      opacity: 0.5;
      z-index: 2000;
      transition: opacity 0.3s;

      &[type="x"]{
        height: 10px;
        left: 0;
        bottom: 0;
        right: 10px;
      }
      &[type="y"]{
        width: 10px;
        right: 0;
        top: 0;
        bottom: 10px;
      }
      .scroll__slider{
        display: block;
        position: absolute;
        border-radius: 5px;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: #C1C1C1;
        &.active, &:hover{
          background: #959595;
        }
      }
    }
    &:hover > .scroll__bar{
      opacity: 1;
    }
  }
</style>
