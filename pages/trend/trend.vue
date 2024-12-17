<template>
  <view class="trend-container">
    <hospital-header></hospital-header>
    <view class="chart-container">
      <!-- uCharts 图表容器 -->
      <qiun-data-charts
        type="line"
        :opts="chartOpts"
        :chartData="chartData"
      />
    </view>
    
    <!-- 数据列表 -->
    <view class="data-list">
      <view class="list-header">
        <text class="header-item">日期</text>
        <text class="header-item">体重(kg)</text>
        <text class="header-item">尿量(ml)</text>
      </view>
      <view class="list-content">
        <view class="list-item" v-for="(item, index) in historyData" :key="index">
          <text class="item-cell">{{formatDate(item.date)}}</text>
          <text class="item-cell">{{item.weight}}</text>
          <text class="item-cell">{{item.urine}}</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      historyData: [],
      chartData: {
        categories: [],
        series: []
      },
      chartOpts: {
        padding: [15, 15, 0, 15],
        legend: {
          show: true,
          position: 'bottom',
          float: 'center'
        },
        xAxis: {
          disableGrid: true
        },
        yAxis: {
          data: [
            {
              position: 'left',
              title: '���重(kg)',
              min: 40,
              max: 100
            },
            {
              position: 'right',
              title: '尿量(ml)',
              min: 0,
              max: 3000
            }
          ]
        },
        extra: {
          line: {
            type: 'straight',
            width: 2
          }
        }
      }
    }
  },
  onLoad() {
    this.loadData()
  },
  onShow() {
    this.loadData()
  },
  methods: {
    loadData() {
      // 获取历史数据
      const data = uni.getStorageSync('waterData') || []
      this.historyData = data.sort((a, b) => new Date(a.date) - new Date(b.date))
      
      // 更新图表数据
      this.updateChartData()
    },
    
    formatDate(dateStr) {
      const date = new Date(dateStr)
      return `${date.getMonth() + 1}/${date.getDate()}`
    },
    
    updateChartData() {
      const categories = this.historyData.map(item => this.formatDate(item.date))
      const weights = this.historyData.map(item => Number(item.weight))
      const urines = this.historyData.map(item => Number(item.urine))
      
      this.chartData = {
        categories: categories,
        series: [
          {
            name: '体重',
            data: weights,
            index: 0,
            type: 'line',
            color: '#5470C6'
          },
          {
            name: '尿量',
            data: urines,
            index: 1,
            type: 'line',
            color: '#91CC75'
          }
        ]
      }
    }
  }
}
</script>

<style>
.trend-container {
  padding: 20px;
}

.chart-container {
  width: 100%;
  height: 400px;
  background: #fff;
  border-radius: 8px;
  margin-bottom: 20px;
}

.data-list {
  background: #fff;
  border-radius: 8px;
  padding: 15px;
}

.list-header {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
  font-weight: bold;
}

.header-item {
  flex: 1;
  text-align: center;
  color: #333;
}

.list-content {
  margin-top: 10px;
}

.list-item {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #f5f5f5;
}

.item-cell {
  flex: 1;
  text-align: center;
  color: #666;
}
</style> 