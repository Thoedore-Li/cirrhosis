<template>
  <view class="trend-container">
    <hospital-header></hospital-header>
    
    <!-- 体重和尿量趋势图 -->
    <view class="chart-container">
      <text class="chart-title">体重和尿量趋势</text>
      <qiun-data-charts
        type="line"
        :opts="chartOpts"
        :chartData="chartData"
      />
    </view>
    
    <!-- 利尿剂使用趋势图 -->
    <view class="chart-container">
      <text class="chart-title">利尿剂使用趋势</text>
      <qiun-data-charts
        type="column"
        :opts="diureticOpts"
        :chartData="diureticData"
      />
    </view>
    
    <!-- 数据列表 -->
    <view class="data-list">
      <view class="list-header">
        <text class="header-item">日期</text>
        <text class="header-item">体重(kg)</text>
        <text class="header-item">尿量(ml)</text>
        <text class="header-item">利尿剂</text>
      </view>
      <view class="list-content">
        <view class="list-item" v-for="(item, index) in historyData" :key="index">
          <text class="item-cell">{{formatDate(item.date)}}</text>
          <text class="item-cell">{{item.weight}}</text>
          <text class="item-cell">{{item.urine}}</text>
          <text class="item-cell">{{formatDiuretics(item.diuretics)}}</text>
        </view>
      </view>
    </view>
    <contact-info></contact-info>
  </view>
</template>

<script>
import ContactInfo from '@/components/contact-info/contact-info.vue'

export default {
  data() {
    return {
      historyData: [],
      chartData: {
        categories: [],
        series: []
      },
      diureticData: {
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
              title: '体重(kg)',
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
      },
      diureticOpts: {
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
              title: '剂量(mg)',
              min: 0
            }
          ]
        },
        extra: {
          column: {
            width: 30,
            activeBgColor: '#2c9f67',
            activeBgOpacity: 0.3
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
      this.updateDiureticData()
    },
    
    formatDate(dateStr) {
      const date = new Date(dateStr)
      return `${date.getMonth() + 1}/${date.getDate()}`
    },
    
    formatDiuretics(diuretics) {
      if (!diuretics || !diuretics.length) return '-'
      return diuretics.map(d => `${d.name}${d.dose}mg`).join('、')
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
    },
    
    updateDiureticData() {
      const categories = this.historyData.map(item => this.formatDate(item.date))
      const diureticTypes = ['螺内酯', '呋塞米', '托伐普坦']
      
      const series = diureticTypes.map(type => {
        return {
          name: type,
          type: 'column',
          data: this.historyData.map(item => {
            const drug = item.diuretics?.find(d => d.name === type)
            return drug ? Number(drug.dose) : 0
          })
        }
      })
      
      this.diureticData = {
        categories,
        series
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
  padding: 20px;
}

.chart-title {
  font-size: 28px;
  color: #333;
  font-weight: bold;
  margin-bottom: 10px;
  display: block;
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