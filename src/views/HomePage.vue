<template>
<!--  <div :class="[multipage === true ? 'multi-page':'single-page', 'not-menu-page', 'home-page']">-->
    <div>
      <img id='image1' src="./logo-blue.png" width="1080px" height="1080px"/>
      <h3 class="title" >{{ msg }}</h3>
      <div class="amap-wrapper">
        <button @click="toggleFullScreen">全屏</button>
<!--        <el-amap class="amap-box" :vid="'amap-vue'"></el-amap>-->
      </div>
  </div>
</template>
<script>
// import HeadInfo from '@/views/common/HeadInfo'
// import {mapState} from 'vuex'
import screenfull from 'screenfull'
import moment from 'moment'
moment.locale('zh-cn')

export default {
  name: 'HomePage',
  data () {
    return {
      isFullScreen: false,
      msg: ''
    }
  },
  computed: {
  },
  methods: {
    toggleFullScreen () {
      if (!screenfull.enabled) {
        alert('浏览器不支持全屏')
        this.msg = '浏览器不支持全屏'
        return false
      }
      var map = document.getElementById('image1')
      console.log(map)
      screenfull.request(map)
    }
  },
  mounted () {
  }
}
</script>
<style lang="less">
  .amap-wrapper {
    width: 1920px;
    height: 1080px;
  }
</style>
