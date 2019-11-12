<template>
<!--  <div :class="[multipage === true ? 'multi-page':'single-page', 'not-menu-page', 'home-page']">-->
  <a-card :bordered="false" class="card-area">
        <div class="operator">
          <a-button type="primary" ghost @click="toggleFullScreen">全屏</a-button>
          <a-button @click="getMapData">刷新</a-button>
          <div style=" float:right">
            <img src="static/img/online.png"/>
            <span>{{this.activeDevice}}</span>
            <img src="static/img/offline.png"/>
            <span>{{this.suspendedDevice}}</span>
          </div>
        </div>
        <div class="card-area" style="text-align: center">
          <el-amap id="amap" class="amap-box" :vid="'amap-vue'" :zoom="zoom" style="width: 100%;height: 420px">
            <el-amap-marker v-for="marker in markers" :key="marker.id" :position="marker.position" :events="marker.events" :icon="marker.icon" :content="marker.content"></el-amap-marker>
            <el-amap-info-window v-if="window" :position="window.position" :visible="window.visible" :content="window.content"></el-amap-info-window>
          </el-amap>
        </div>
  </a-card>
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
  name: 'NumberMap',
  data () {
    // let self = this
    return {
      position: [118.08944444, 24.46583333],
      isFullScreen: false,
      initMapWidth: 0,
      zoom: 14,
      markers: [],
      windows: [],
      window: '',
      mapData: [],
      activeDevice: '在线:0',
      suspendedDevice: '离线:0'
    }
  },
  mounted () {
    this.getMapData()
    window.onresize = () => {
      // 全屏下监控是否按键了ESC
      if (!this.checkFull()) {
        // 全屏下按键esc后要执行的动作
        // 全屏
        var map = document.getElementById('amap-vue')
        map.style.width = this.initMapWidth + 'px'
        // 设为原来的尺寸
        map.style.height = 420 + 'px'
        this.isFullScreen = false
      }
    }
  },
  methods: {
    toggleFullScreen () {
      if (!screenfull.enabled) {
        alert('浏览器不支持全屏')
        this.msg = '浏览器不支持全屏'
        return false
      }
      // 全屏
      let map = document.getElementById('amap-vue')
      if (this.initMapWidth === 0) {
        this.initMapWidth = map.clientWidth
      }
      map.style.width = 1920 + 'px'
      map.style.height = 1080 + 'px'
      screenfull.toggle(map)
      this.isFullScreen = true
    },
    /**
     * 是否全屏并按键ESC键的方法
     */
    checkFull () {
      let isFull = document.fullscreenEnabled || window.fullScreen || document.webkitIsFullScreen || document.msFullscreenEnabled
      // to fix : false || undefined == undefined
      if (isFull === undefined) {
        isFull = false
      }
      return isFull
    },
    // 获取设备数据
    getMapData () {
      this.$get(`number/mapList`).then((r) => {
        this.mapData = r.data
        this.createMarkers()
        this.renderStats()
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
        let imgUrl = '"' + 'http://106.13.22.160/group1/' + item.url + '"'
        // let imgUrl = '../../static/img/logo.
        if (item.status === '1') {
          // content = '<img src="../../static/icons/pos1.jpeg" class="active-marker">'
          content = '<img src=' + imgUrl + ' class="active-marker">'
        } else if (item.status === '0') {
          content = '<img src=' + imgUrl + ' class="suspended-marker">'
        }
        console.log(content)
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
    },
    renderStats () {
      let mapData = this.mapData
      let length = mapData.length
      let activeDevice = 0
      let suspendedDevice = 0
      for (let i = 0; i < length; i++) {
        if (mapData[i].status === '1') {
          activeDevice++
        } else {
          suspendedDevice++
        }
      }
      this.activeDevice = '在线:' + activeDevice
      this.suspendedDevice = '离线:' + suspendedDevice
    }
  }
}
</script>
<style lang="less">
  @import "../../../../static/less/Common";
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
