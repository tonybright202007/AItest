<template>
  <div class="dashboard">
    <header class="header">
      <h1>销售分析看板</h1>
      <div class="date-range">
        <span>日期范围：</span>
        <select v-model="selectedRange">
          <option value="7">近7天</option>
          <option value="30">近30天</option>
          <option value="90">近90天</option>
        </select>
      </div>
    </header>
    
    <div class="stats-cards">
      <div class="stat-card">
        <div class="stat-icon sales-icon">💰</div>
        <div class="stat-content">
          <div class="stat-label">总销售额</div>
          <div class="stat-value">¥{{ formatNumber(totalSales) }}</div>
          <div class="stat-change positive">↑ {{ salesGrowth }}%</div>
        </div>
      </div>
      <div class="stat-card">
        <div class="stat-icon order-icon">📦</div>
        <div class="stat-content">
          <div class="stat-label">订单数量</div>
          <div class="stat-value">{{ totalOrders }}</div>
          <div class="stat-change positive">↑ {{ orderGrowth }}%</div>
        </div>
      </div>
      <div class="stat-card">
        <div class="stat-icon customer-icon">👥</div>
        <div class="stat-content">
          <div class="stat-label">客户数量</div>
          <div class="stat-value">{{ totalCustomers }}</div>
          <div class="stat-change positive">↑ {{ customerGrowth }}%</div>
        </div>
      </div>
      <div class="stat-card">
        <div class="stat-icon avg-icon">📊</div>
        <div class="stat-content">
          <div class="stat-label">客单价</div>
          <div class="stat-value">¥{{ avgOrderValue }}</div>
          <div class="stat-change negative">↓ {{ avgGrowth }}%</div>
        </div>
      </div>
    </div>
    
    <div class="chart-section">
      <div class="chart-card">
        <h2>销售趋势</h2>
        <div ref="chartRef" class="chart"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import * as echarts from 'echarts'

const selectedRange = ref('30')
const chartRef = ref(null)
let chartInstance = null

const generateMockData = (days) => {
  const data = []
  const today = new Date()
  for (let i = days - 1; i >= 0; i--) {
    const date = new Date(today)
    date.setDate(date.getDate() - i)
    const dateStr = `${date.getMonth() + 1}/${date.getDate()}`
    const baseValue = 10000 + Math.random() * 5000
    const weekendBonus = (date.getDay() === 0 || date.getDay() === 6) ? Math.random() * 3000 : 0
    data.push({
      date: dateStr,
      sales: Math.round(baseValue + weekendBonus),
      orders: Math.round((baseValue + weekendBonus) / 150)
    })
  }
  return data
}

const mockData = computed(() => generateMockData(parseInt(selectedRange.value)))

const totalSales = computed(() => {
  return mockData.value.reduce((sum, item) => sum + item.sales, 0)
})

const totalOrders = computed(() => {
  return mockData.value.reduce((sum, item) => sum + item.orders, 0)
})

const totalCustomers = computed(() => {
  return Math.round(totalOrders.value * 0.8)
})

const avgOrderValue = computed(() => {
  return Math.round(totalSales.value / totalOrders.value)
})

const calculateGrowth = (data) => {
  const recent = data.slice(-7)
  const previous = data.slice(-14, -7)
  const recentSum = recent.reduce((sum, item) => sum + item.sales, 0)
  const previousSum = previous.reduce((sum, item) => sum + item.sales, 0)
  if (previousSum === 0) return 0
  return ((recentSum - previousSum) / previousSum * 100).toFixed(1)
}

const salesGrowth = computed(() => calculateGrowth(mockData.value))
const orderGrowth = computed(() => {
  const recent = mockData.value.slice(-7)
  const previous = mockData.value.slice(-14, -7)
  const recentSum = recent.reduce((sum, item) => sum + item.orders, 0)
  const previousSum = previous.reduce((sum, item) => sum + item.orders, 0)
  if (previousSum === 0) return 0
  return ((recentSum - previousSum) / previousSum * 100).toFixed(1)
})

const customerGrowth = computed(() => {
  return (Math.random() * 5).toFixed(1)
})

const avgGrowth = computed(() => {
  return (Math.random() * 2).toFixed(1)
})

const formatNumber = (num) => {
  return num.toLocaleString('zh-CN')
}

const initChart = () => {
  if (!chartRef.value) return
  
  chartInstance = echarts.init(chartRef.value)
  updateChart()
}

const updateChart = () => {
  if (!chartInstance) return
  
  const option = {
    tooltip: {
      trigger: 'axis',
      formatter: '{b}<br/>销售额: ¥{c}',
      backgroundColor: 'rgba(255, 255, 255, 0.95)',
      borderColor: '#e0e0e0',
      borderWidth: 1,
      textStyle: {
        color: '#333'
      }
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      top: '10%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: mockData.value.map(item => item.date),
      axisLine: {
        lineStyle: {
          color: '#eee'
        }
      },
      axisLabel: {
        color: '#666',
        fontSize: 12
      }
    },
    yAxis: {
      type: 'value',
      axisLine: {
        show: false
      },
      axisTick: {
        show: false
      },
      splitLine: {
        lineStyle: {
          color: '#f0f0f0'
        }
      },
      axisLabel: {
        color: '#666',
        formatter: (value) => {
          if (value >= 10000) {
            return (value / 10000).toFixed(1) + '万'
          }
          return value
        }
      }
    },
    series: [
      {
        name: '销售额',
        type: 'line',
        smooth: true,
        data: mockData.value.map(item => item.sales),
        lineStyle: {
          width: 3,
          color: '#5470c6'
        },
        itemStyle: {
          color: '#5470c6'
        },
        areaStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: 'rgba(84, 112, 198, 0.3)' },
            { offset: 1, color: 'rgba(84, 112, 198, 0.05)' }
          ])
        },
        symbol: 'circle',
        symbolSize: 6
      }
    ]
  }
  
  chartInstance.setOption(option)
}

const handleResize = () => {
  chartInstance?.resize()
}

onMounted(() => {
  initChart()
  window.addEventListener('resize', handleResize)
})

watch(selectedRange, () => {
  updateChart()
})
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

.dashboard {
  padding: 20px;
  max-width: 1400px;
  margin: 0 auto;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  color: white;
}

.header h1 {
  font-size: 28px;
  font-weight: 600;
}

.date-range {
  display: flex;
  align-items: center;
  gap: 10px;
  background: rgba(255, 255, 255, 0.2);
  padding: 8px 16px;
  border-radius: 20px;
}

.date-range select {
  background: white;
  border: none;
  padding: 6px 12px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
}

.stats-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 16px;
  margin-bottom: 20px;
}

.stat-card {
  background: white;
  border-radius: 16px;
  padding: 20px;
  display: flex;
  align-items: center;
  gap: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.stat-icon {
  width: 50px;
  height: 50px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
}

.sales-icon {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.order-icon {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
}

.customer-icon {
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
}

.avg-icon {
  background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);
}

.stat-content {
  flex: 1;
}

.stat-label {
  font-size: 14px;
  color: #999;
  margin-bottom: 4px;
}

.stat-value {
  font-size: 24px;
  font-weight: 600;
  color: #333;
  margin-bottom: 4px;
}

.stat-change {
  font-size: 12px;
  font-weight: 500;
}

.stat-change.positive {
  color: #52c41a;
}

.stat-change.negative {
  color: #f5222d;
}

.chart-section {
  background: white;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.chart-card h2 {
  font-size: 18px;
  font-weight: 600;
  color: #333;
  margin-bottom: 20px;
}

.chart {
  width: 100%;
  height: 400px;
}
</style>
