<template>
<!--  <div :class="[multipage === true ? 'multi-page':'single-page', 'not-menu-page', 'home-page']">-->
    <div>
<!--      <img id='image1' src="./logo-blue.png" width="1080px" height="1080px"/>-->
      <h3 class="title" id="hello" >{{ msg }}</h3>
      <div class="amap-wrapper">
        <button @click="add">add marker</button>
        <button @click="toggleFullScreen">全屏</button>
        <el-amap id="amap" class="amap-box" :vid="'amap-vue'" :zoom="zoom" :amap-manager="amapManager" :events="events">
          <el-amap-marker v-for="marker in markers" :position="marker.position" :events="marker.events"></el-amap-marker>
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
// eslint-disable-next-line import/no-duplicates
// import { AMapManager } from 'vue-amap';
import VueAMap from 'vue-amap'
let amapManager = new VueAMap.AMapManager()
moment.locale('zh-cn')
export default {
  name: 'HomePage',
  data () {
    // let self = this
    return {
      position: [118.08944444, 24.46583333],
      isFullScreen: false,
      msg: 'jdjdjdj',
      zoom: 14,
      markers: [],
      windows: [],
      window: '',
      mapData: [],
      // center: [121.5273285, 31.21515044],
      events: {
        // init (o) {
        // let marker = new AMap.Marker({
        //   position: [118.08944444, 24.46583333]
        // })
        // marker.setMap(o)
        // marker.setLabel({
        //   offset: new AMap.Pixel(15, 15),
        //   content: '定位名称'
        // })
        // var swBound = new AMap.LngLat([117.08944444, 23.46583333])
        // var neBound = new AMap.LngLat([119.08944444, 25.46583333])
        // var bounds = new AMap.Bounds(swBound, neBound)
        // o.setBounds(bounds)
        // var layer = new AMap.TileLayer({
        //   zooms: [3, 20],
        //   visible: false,
        //   opacity: 1,
        //   zIndex: 0
        // })
        // o.add(layer)
        // layers.setMap(o)
        init (map) {
          // AMapUI.loadUI(['overlay/SimpleMarker'], function (SimpleMarker) {
          //   const marker = new SimpleMarker({
          //     iconLabel: 'A',
          //     iconStyle: 'red',
          //     map: map,
          //     position: map.getCenter()
          //   })
          // })
        }
      }
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
      var map = document.getElementById('amap-vue')
      map.style.width = 1920 + 'px'
      map.style.height = 1080 + 'px'
      screenfull.toggle(map)
    },
    add () {
      let o = amapManager.getMap()
      let marker = new AMap.Marker({
        position: [140, 40]
      })
      marker.setMap(o)
    },
    getMapData () {
      let categoryId = 0
      this.$get(`number/map/${categoryId}`).then((r) => {
        this.mapData = r.data
        console.log(this.mapData)
        let markers = []
        let windows = []
        let self = this
        let mapData = this.mapData
        // var count = 0
        for (let i = 0; i < mapData.length; i++) {
          let item = mapData[i]
          if (item.latitudes && item.longitudes) {
            let position = [item.longitudes, item.latitudes]
            console.log('position', position)
            markers.push({
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
              },
              icon: new AMap.Icon({
                image: '../../../icons/pos.jpg',
                size: new AMap.Size(52, 52), // 图标大小
                imageSize: new AMap.Size(26, 26)
              })
            })
            windows.push({
              position: position,
              content: `<div>编号：${item.code}</div><div>部署位置：${item.location}</div>`,
              visible: false
            })

          }
        }
        this.markers = markers
        this.windows = windows
      })
    }
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
</style>
