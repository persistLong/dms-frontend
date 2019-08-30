<template>
<!--  <div :class="[multipage === true ? 'multi-page':'single-page', 'not-menu-page', 'home-page']">-->
    <div>
<!--      <img id='image1' src="./logo-blue.png" width="1080px" height="1080px"/>-->
      <div class="amap-wrapper">
        <button @click="toggleFullScreen" class="fullscreen-button">全屏</button>
        <el-amap id="amap" class="amap-box" :vid="'amap-vue'" :zoom="zoom">
          <el-amap-marker v-for="marker in markers" :key="marker.id" :position="marker.position" :events="marker.events" :icon="marker.icon" :content="marker.content"></el-amap-marker>
          <el-amap-info-window v-if="window" :position="window.position" :visible="window.visible" :content="window.content"></el-amap-info-window>
        </el-amap>
      </div>
  </div>
</template>
<script>
import screenfull from 'screenfull'
import moment from 'moment'
// import HeadInfo from '@/views/common/HeadInfo'
// import {mapState} from 'vuex
// eslint-disable-next-line import/no-duplicatesc
// import { AMapManager } from 'vue-amap';
// import AMap from 'amap'
import VueAMap from 'vue-amap'
// eslint-disable-next-line no-unused-vars
let amapManager = new VueAMap.AMapManager()
moment.locale('zh-cn')
export default {
  name: 'HomePage',
  data () {
    // let self = this
    return {
      position: [118.08944444, 24.46583333],
      isFullScreen: false,
      zoom: 14,
      markers: [],
      windows: [],
      window: '',
      mapData: [],
      activeDevice: 0,
      suspendedDevice: 0
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
      // 全屏
      var map = document.getElementById('amap-vue')
      map.style.width = 1920 + 'px'
      map.style.height = 1080 + 'px'
      screenfull.toggle(map)
    },
    getMapData () {
      let categoryId = 0
      this.$get(`number/map/${categoryId}`).then((r) => {
        this.mapData = r.data
        this.createMarkers()
      })
    },
    createMarkers () {
      let markers = []
      let windows = []
      let self = this
      let mapData = this.mapData
      // 只渲染有经纬度的设备, 过滤掉没有经纬度的设备
      let filterMap = []
      for (let i = 0; i < mapData.length; i++) {
        let item = mapData[i]
        if (item.latitudes && item.longitudes) {
          filterMap.push(item)
        }
      }
      for (let i = 0; i < filterMap.length; i++) {
        let item = filterMap[i]

        let position = [item.longitudes, item.latitudes]
        // 根据状态设置图标边框样式
        let content = ''
        // let imgUrl = 'http://192.168.179.55/group1/' + item.url
        // let imgUrl = '../../static/img/logo.
        if (item.status === '1') {
          content = '<img src="../../static/icons/pos1.jpeg" class="active-marker">'
        } else if (item.status === '0') {
          content = '<img src="../../static/icons/pos1.jpeg" class="suspended-marker">'
        }
        markers.push({
          content: content,
          position: position,
          events: {
            click () {
              self.windows.forEach(window => {
                window.visible = false
              })
              self.window = self.windows[i]
              self.$nextTick(() => {
                self.window.visible = true
              })
            }
          }
        })
        windows.push({
          position: position,
          content: `<div>编号：${item.code}</div><div>部署位置：${item.location}</div>`,
          visible: false
        })
      }

      this.markers = markers
      this.windows = windows
    }
  },
  renderStats () {
    let mapData = this.mapData
    let length = mapData.length
    let activeDevice = 0
    let suspendedDevice = 0
    for (var i = 0; i < length; i++) {
      if (mapData[i].status) {
        activeDevice++
      } else {
        suspendedDevice++
      }
    }
    this.activeDevice = activeDevice
    this.suspendedDevice = suspendedDevice
  },
  mounted () {
    this.getMapData()
  }
}
</script>
<style lang="less">
  .amap-wrapper {
    width: 1920px;
    height: 1080px;
  }
  .active-marker{
    border-radius: 50%;
    border: green solid 2px;
  }
  .suspended-marker{
    border-radius: 50%;
    border: #cf1322 solid 2px;
  }
  .fullscreen-button{
    /*float: right;*/
  }
</style>
