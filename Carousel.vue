<!-- 
  date: 2018-06-08
  author: wanlixin
  desc: carousel component use image or video, Stop after the finger touch and automatically roll after loosening
  caution: The CSS part uses REM units, which can be NPM I --S flexible.js, and then in main.js file. 
    `import flexible from './assets/flexible.js'
     flexible(750, 640)
    `
  params: {
    item: {
      BANNER_URL:  The path of media resources
      MEDIA_TYPE:  0 for picture 1 for video
    }
    props: {
      bannerList: Array,
      pager: Boolean,
      pargerWidth: Number  The percentage of the width of the pager
      pagerPosition: {
        VerticalDirection: 'bottom',
        VerticalNumber: 30,
        HorizontalDirection: 'right',
        HorizontalNumber: 30,
        HorizontalCenter: true   horizontally
      }
      mediaHeight: Number  The height unit of media resources is defaults to 'PX',
      duration:  The default time of the rolling interval is 2000ms
    },
    event: {
      clickSlideHandle: Click on the response of a media response,
      exitfullscreen: Full screen,
      enterfullscreen:  Exit full screen
    }
  }
 -->
<template>
  <div 
    class="wan_swiper_container"
    @touchstart="touchstartHandle"
    @touchmove="touchmoveHandle"
    @touchend="touchendHandle"
  >
    <div class="wan_swiper_wrapper"
      :style="{
        width: wrapperWidth, minWidth: widthScreen * bannerList.length + 'rem',
        transform: wrapperTransform
      }"
    >
      <div class="wan_swiper_slide"
        v-for="(item, index) in bannerList"
        :key="item.SORT"
        :class="{
          'current_slide': current === index,
          'sbiling_slide': current === index - 1 || current === index + 1,
        }"
        :style="{
          width: widthScreen + 'rem',
          height: (mediaHeight / rem) + 'rem'
        }"
      >
        <p @click="$emit('clickSlideHandle', item)">
          <video class="my-swiper-video" 
            v-if="item.MEDIA_TYPE == 1" 
            :src="item.BANNER_URL"
            preload="preload"
            x5-video-player-type="h5"
            x5-video-player-fullscreen="true"
            controls="controls"
            poster="http://cntaipingapp.oss-cn-hangzhou.aliyuncs.com/oldage/picture/banner/vedio_andriod.jpg"
            @x5videoexitfullscreen="$emit('exitfullscreen')"
            @x5videoenterfullscreen="$emit('enterfullscreen')"
          ></video>
          <img v-if="item.MEDIA_TYPE == 0" :src="item.BANNER_URL" />
        </p>
      </div>
    </div>
    <div class="my_swiper_pager" 
      v-if="pager"
      :style="{
        width: pargerWidth + '%',
        [pagerPosition.VerticalDirection]: (pagerPosition.VerticalNumber/rem) + 'rem',
        [pagerPosition.HorizontalDirection]: pagerPosition.HorizontalCenter ? (100 - pargerWidth) / 2 + '%' : (pagerPosition.HorizontalNumber/rem) + 'rem'
      }"
    >
      <span 
        v-for="(item, index) in bannerList.length"
        :key="index"
        @click="clickPage(index)"
        class="page"
        :style="{background: current === index ? pagerColor : ''}"></span>
    </div>
  </div>
</template>
<script>
  export default {
    data () {
      return {
        selected: 0,
        wrapperWidth: 0,
        sx: 0,
        sy: 0,
        ex: 0,
        ey: 0,
        dx: 0,
        dy: 0,
        lx: 50,
        ly: 80,
        current: 0,
        wrapperTransform: '',
        vrLink: '',
        widthScreen: '',
        rem: 50
      }
    },
    props: {
      bannerList: [Array],
      pager: {
        type: Boolean,
        default: true
      },
      pagerColor: {
        type: String,
        default: 'green'
      },
      pargerWidth: {
        type: Number,
        default: 40
      },
      pagerPosition: [Object],
      mediaHeight: [Number],
      duration: {
        type: Number,
        default: 2000
      },
    },
    mounted () {
      this.widthScreen = (document.body.clientWidth) / this.rem;
      this.interval();
    },
    updated () {
      this.wrapperWidth = (this.widthScreen * this.bannerList.length) / this.rem + 'rem';
    },
    methods: {
      touchstartHandle (e) {
        e = e || window.event;
        this.sx = e.touches[0].pageX;
        this.sy = e.touches[0].pageY;
        clearInterval(this.timmer);
      },
      touchmoveHandle (e) {
        e = e || window.event;
        this.ex = e.touches[0].pageX;
        this.ey = e.touches[0].pageY;
        clearInterval(this.timmer);
      },
      touchendHandle () {
        this.dx = this.ex - this.sx;
        this.dy = this.ey - this.sy;
        // Slide to the left
        if (this.ex && Math.abs(this.dy) < this.ly && Math.abs(this.dx) > this.lx && this.dx < -this.lx) {
          this.current++;
          this.current > this.bannerList.length - 1 && (this.current = 0)
          this.swiper()
        }

        // Slide to the right
        if (this.ex && Math.abs(this.dy) < this.ly && Math.abs(this.dx) > this.lx && this.dx > this.lx) {
          this.current--;
          this.current < 0 && (this.current = this.bannerList.length - 1)
          this.swiper()
        }
        this.interval();
        this.dx = this.sx = this.ex = 0;
        this.dy = this.sy = this.ey = 0;

      },
      swiper () {
        this.wrapperTransform = `translateX(-${this.current * this.widthScreen}rem)`
      },
      clickPage (index) {
        this.current = index;
        this.swiper();
      },
      interval () { // autoplay
        this.timmer = setInterval( () => {
          this.current++;
          if (this.current > this.bannerList.length - 1) this.current = 0;
          this.swiper()
        }, this.duration)
      }
    }
  }
</script>
<style scoped>
  .wan_swiper_container {
    z-index: 9;
    position: relative;
    height: auto;
    width: 100%;
    overflow: hidden;
  }
  .wan_swiper_wrapper{
    height: auto;
    overflow: hidden;
  }
  .wan_swiper_slide{
    float: left;
    opacity: 0.4;
  }
  .current_slide {
    transition: opacity 0.3s ease-in;
    opacity: 1;
  }
  .sbiling_slide{
    transition: opacity 0.3s ease-in-out;
    opacity: 0.4;
  }
  .my_swiper_pager {
    position: absolute;
    display: flex;
    justify-content: space-between;
    height: .2rem/* 10px */
  }
  .page{
    width: .4rem/* 20px */;
    height: .1rem/* 5px */;
    border-radius: .04rem/* 2px */;
    background: #fff;
  }
  /*.paging{
    background: green;
  }*/
  video,img {
    width: 100%;
    vertical-align: middle;
  }
</style>