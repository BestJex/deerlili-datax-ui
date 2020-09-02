<template>
  <div :class="className" :style="{height:height,width:width}" />
</template>

<script>
import echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import resize from './mixins/resize'
import { getList } from '@/api/datax-registry'

const animationDuration = 6000

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '300px'
    }
  },
  data() {
    return {
      chart: null
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initChart()
    })
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    fetchData() {
      this.listLoading = true
      this.list = []
      getList(this.listQuery).then(response => {
        const { records } = response
        const { total } = response
        this.total = total
        this.list = records
        this.listLoading = false
        this.$nextTick(function() {
          for (let i = 0; i < this.list.length; i++) {
            this.initChart(this.list[i])
          }
        })
      })
    },
    initChart() {
      this.chart = echarts.init(this.$el, 'macarons')

      this.chart.setOption({
        tooltip: {
          trigger: 'axis',
          axisPointer: { // 坐标轴指示器，坐标轴触发有效
            type: 'line' // 默认为直线，可选为：'line' | 'shadow'
          }
        },
        grid: {
          top: 10,
          left: '2%',
          right: '2%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [{
          type: 'category',
          data: ['192.168.8.118:9999', '192.168.8.119:9999'],
          axisTick: {
            alignWithLabel: true
          }
        }],
        yAxis: [{
          type: 'value',
          axisTick: {
            show: false
          }
        }],
        series: [{
          name: 'CPU使用率',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: [40, 60, 200],
          animationDuration
        }, {
          name: '内存使用率',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: [40, 50, 200],
          animationDuration
        }, {
          name: '负载',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: [2, 1, 0],
          animationDuration
        }]
      })
    }
  }
}
</script>
