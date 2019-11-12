<template>
  <div :class="[multipage === true ? 'multi-page':'single-page', 'not-menu-page', 'home-page']">
    <a-row :gutter="8" class="count-info">
      <a-col :span="12" class="visit-count-wrapper">
        <a-card class="visit-count">
          <apexchart type=pie width=380 height=400 :options="chartOptions" :series="series" />
          <div style="text-align: center">设备在线情况统计</div>
        </a-card>
      </a-col>
      <a-col :span="12" class="visit-count-wrapper">
        <a-card class="visit-count">
          <apexchart type=line width=380 height=400 :options="chartOptions1" :series="series1" />
          <div style="text-align: center">设备故障日志情况统计</div>
        </a-card>
      </a-col>
    </a-row>
  </div>
</template>
<script>
import moment from 'moment'
moment.locale('zh-cn')

export default {
  name: 'HomePage',
  data () {
    return {
      series: [],
      chartOptions: {
        labels: ['在线设备', '离线设备'],
        responsive: [{
          breakpoint: 480,
          options: {
            chart: {
              width: 200
            },
            legend: {
              position: 'bottom'
            }
          }
        }]
      },
      series1: [{
        name: 'Log',
        data: []
      }],
      chartOptions1: {
        chart: {
          height: 350,
          zoom: {
            enabled: false
          }
        },
        dataLabels: {
          enabled: false
        },
        stroke: {
          curve: 'straight'
        },
        grid: {
          row: {
            colors: ['#f3f3f3', 'transparent'], // takes an array which will be repeated on columns
            opacity: 0.5
          }
        },
        xaxis: {
          categories: []
        }
      }
    }
  },
  methods: {
    // 获取设备数据
    getMapData () {
      this.series1[0].data = []
      this.$get(`number/mapList`).then((r) => {
        let mapData = r.data
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
        this.series.push(activeDevice)
        this.series.push(suspendedDevice)
        console.info(this.series)
      })
    },
    spitDateStr (date) {
      let strs = [] // 定义一数组
      strs = date.split(' ') // 字符分割
      return strs[0]
    },
    getDateStr (AddDayCount) {
      let dd = new Date()
      dd.setDate(dd.getDate() + AddDayCount)// 获取AddDayCount天后的日期
      let y = dd.getFullYear()
      let m = dd.getMonth() + 1// 获取当前月份的日期
      let d = dd.getDate()
      return y + '-' + (m < 10 ? '0' + m : m) + '-' + (d < 10 ? '0' + d : d)
    },
    getHitchData () {
      this.series1[0].data = []
      this.chartOptions1.xaxis.categories = []
      this.$get(`hitch/hitchList`).then((r) => {
        let hitchData = r.data
        let length = hitchData.length
        let values = [0, 0, 0, 0, 0, 0, 0]
        let dateArray = [this.getDateStr(-6), this.getDateStr(-5), this.getDateStr(-4),
          this.getDateStr(-3), this.getDateStr(-2), this.getDateStr(-1), this.getDateStr(0)]
        console.info(dateArray)
        for (let i = 0; i < length; i++) {
          for (let j = 0; j < dateArray.length; j++) {
            if (dateArray[j] === this.spitDateStr(hitchData[i].createTime)) {
              values[j]++
            }
          }
        }
        this.series1[0].data = values
        this.chartOptions1.xaxis.categories = dateArray
      })
    }
  },
  mounted () {
    this.getMapData()
    this.getHitchData()
  }
}
</script>
<style lang="less">
    .count-info {
      .visit-count-wrapper {
        padding-left: 0 !important;
        .visit-count {
          padding: .5rem;
          border-color: #f1f1f1;
          .ant-card-body {
            padding: .5rem 1rem !important;
          }
        }
      }
    }
    #chart {
      max-width: 380px;
      margin: 35px auto;
    }

</style>
