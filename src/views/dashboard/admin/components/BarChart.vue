<template>
  <div class="app-container">
    <!-- <div class="filter-container">
      <el-input v-model="listQuery.registryKey" placeholder="执行器" style="width: 200px;" class="filter-item" />
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="fetchData">
        Search
      </el-button>
    </div> -->
    <div v-for="item in list" :key="item.id" class="container">
      <!-- <div>执行器：{{ item.registryKey }}</div> -->
      <!-- <el-row>
        <el-col :span="12">注册地址：{{ item.registryValue }}</el-col>
        <el-col :span="12">更新时间：{{ item.updateTime | parseTime('{y}-{m}-{d} {h}:{i}') }}
        </el-col>
      </el-row> -->
      <el-row>
        <el-col :span="13">注册地址：{{ item.registryValue }}</el-col>
        <el-col :span="11">更新时间：{{ item.updateTime | parseTime('{m}-{d} {h}:{i}') }}
        </el-col>
      </el-row>
      <br>
      <div :class="item.id + ' fl'" style="width: 50%;height: 270px;color:#da5416;" />
      <div :class="item.id + ' fl'" style="width: 50%;height: 270px;color:#da5416;" />
    </div>
  </div>
</template>

<script>
import { getList } from '@/api/datax-registry'
import waves from '@/directive/waves'
import parseTime from '@/utils'

export default {
  name: 'Registry',
  directives: { waves, parseTime },
  data() {
    return {
      list: null,
      listLoading: true,
      total: 0,
      listQuery: {
        current: 1,
        size: 10,
        registryKey: undefined
      },
      dialogPluginVisible: false
    }
  },
  created() {
    this.fetchData()
  },
  mounted() {
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
            this.initEcharts(this.list[i])
          }
        })
      })
    },
    initEcharts(data) {
      const myChart1 = this.$echarts.init(document.getElementsByClassName(data.id)[0])
      // 绘制图表
      var option1 = {
        title: {
          text: 'cpu使用率',
          subtext: '',
          x: 'center'
        },
        tooltip: {
          formatter: '{a}: {c}%'
        },
        toolbox: {
          feature: {
            restore: {},
            saveAsImage: {}
          },
          show: false
        },
        series: [{
          name: 'cpu使用率',
          type: 'gauge',
          max: 100,
          radius: '80%', // 半径
          startAngle: 215, // 起始位置
          endAngle: -35, // 终点位置
          detail: {
            formatter: '{value}%'
          },
          data: [{
            value: data.cpuUsage,
            name: ''
          }]
        }]
      }
      myChart1.setOption(option1)

      const myChart2 = this.$echarts.init(document.getElementsByClassName(data.id)[1])
      // 绘制图表
      var option2 = {
        title: {
          text: '内存使用率',
          subtext: '',
          x: 'center'
        },
        tooltip: {
          formatter: '{a}: {c}%'
        },
        toolbox: {
          feature: {
            restore: {},
            saveAsImage: {}
          },
          show: false
        },
        series: [{
          name: '内存使用率',
          type: 'gauge',
          max: 100,
          radius: '80%', // 半径
          startAngle: 215, // 起始位置
          endAngle: -35, // 终点位置
          detail: {
            formatter: '{value}%'
          },
          data: [{
            value: data.memoryUsage,
            name: ''
          }]
        }]
      }
      myChart2.setOption(option2)
    }
  }
}
</script>
<style lang="scss" scope>
  .container{
    overflow: hidden;
    p{
      font-size: 12px;color: rgba(0, 0, 0, 0.45);padding: 10px 0;
      .fl{
        float: left;font-size: 12px;
      }
      .fr{
        float: right;
      }
    }
  }
</style>
