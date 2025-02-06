<template>
  <div id="myChart" class="total-class" :style="{width: '100%', height: '400px'}" />
</template>

<script>
import echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme

export default {
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
      default: '350px'
    },
    autoResize: {
      type: Boolean,
      default: true
    },
    chartData: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      chart: null
    }
  },
  watch: {
    chartData: {
      deep: true,
      handler(val) {
        this.setOptions(val)
      }
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
    window.removeEventListener('resize', this.handleResize)
  },
  methods: {
    initChart() {
      this.chart = echarts.init(this.$el, 'macarons')
      this.setOptions(this.chartData)
      window.addEventListener('resize', this.handleResize)
    },
    handleResize() {
      if (this.chart) {
        this.chart.resize()
      }
    },
    setOptions({ hatCount, personCount } = {}) {
      this.chart.setOption({
        xAxis: {
          data: ['批次1', '批次2', '批次3', '批次4', '批次5', '批次6', '批次7', '批次8', '批次9', '批次10'],
          boundaryGap: false,
          axisTick: {
            show: false
          }
        },
        grid: {
          left: 10,
          right: 10,
          bottom: 20,
          top: 30,
          containLabel: true
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross'
          },
          padding: [5, 10]
        },
        yAxis: {
          axisTick: {
            show: false
          },
          name: '数量（个）'
        },
        legend: {
          data: ['合格零件', '缺陷零件']
        },
        series: [{
          name: '合格零件', 
          itemStyle: {
            normal: {
              color: '#ff1aff',
              lineStyle: {
                color: '#ff1aff',
                width: 2
              }
            }
          },
          smooth: true,
          type: 'line',
          data: hatCount,
          animationDuration: 2800,
          animationEasing: 'cubicInOut'
        },
        {
          name: '缺陷零件',
          smooth: true,
          type: 'line',
          itemStyle: {
            normal: {
              color: '#00ffcc',
              lineStyle: {
                color: '#00ffcc',
                width: 2
              }
            }
          },
          data: personCount,
          animationDuration: 2800,
          animationEasing: 'quadraticOut'
        }]
      })
    }
  }
}
</script>