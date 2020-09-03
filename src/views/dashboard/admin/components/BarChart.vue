<template>
  <div class="app-container">
    <!-- <div class="filter-container">
      <el-input v-model="listQuery.registryKey" placeholder="执行器" style="width: 200px;" class="filter-item" />
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="fetchData">
        Search
      </el-button>
    </div> -->
    <div v-for="item in list" :key="item.id" class="container">
      <div style="text-align:center;"><el-tag style="color:#da5416;font-size: 18px;">执行器：{{ item.registryKey }}</el-tag></div>
      <el-row>
        <!-- <el-col :span="12"><el-tag>注册地址：{{ item.registryValue }}</el-tag></el-col>
        <el-col :span="12"><el-tag>更新时间：{{ item.updateTime | parseTime('{y}-{m}-{d} {h}:{i}') }}</el-tag>
        </el-col> -->
      </el-row>
      <el-tag>注册地址：{{ item.registryValue }}</el-tag>
      <el-tag>更新时间：{{ item.updateTime }}</el-tag>
      <br>
      <div :class="item.id + ' fl'" style="width: 50%;height: 200px;color:#da5416;" />
      <div :class="item.id + ' fl'" style="width: 50%;height: 200px;color:#da5416;" />
      <!-- <div :class="item.id + ' fl' + ' loadAverage'" style="width: 30%;height: 300px ">
        <p class="title">平均负载</p>
        <p class="number">{{ item.loadAverage >= 0 ? item.loadAverage : 0 }}</p>
      </div> -->

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
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
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
          radius: '100%', // 半径
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
          radius: '100%', // 半径
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
      font-size: 14px;color: rgb(102, 102, 102);padding: 10px 0;
      .fl{
        float: left;
      }
      .fr{
        float: right;
      }
    }
    .loadAverage{
      p{
        text-align: center;
      }
      .title{
        font-size: 18px;font-weight: bold;color: #333;padding: 5px 0;margin: 0;
      }
      .number{
        font-size: 50px;color: #da5416
      }
    }
  }
</style>
