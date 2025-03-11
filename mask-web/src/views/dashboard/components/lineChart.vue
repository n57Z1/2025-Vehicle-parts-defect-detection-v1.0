<template>
  <div :class="className" :style="{height:height,width:width}" />
</template>

<script>
import * as echarts from 'echarts'

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
  mounted() {
    this.initChart()
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    initChart() {
      this.chart = echarts.init(this.$el)
      
      const hours = Array.from({length: 24}, (_, i) => `${i}:00`)
      
      this.setOptions(hours)
    },
    setOptions(hours) {
      const option = {
        backgroundColor: 'transparent',
        title: {
          text: '24小时缺陷监控详情',
          left: 'center',
          textStyle: {
            color: '#eee',
            fontSize: 20,
            fontWeight: 600
          },
          padding: [20, 0]
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'line',
            lineStyle: {
              color: 'rgba(255, 255, 255, 0.3)',
              width: 1,
              type: 'solid'
            }
          },
          backgroundColor: 'rgba(0, 0, 0, 0.8)',
          borderColor: 'rgba(255, 255, 255, 0.3)',
          borderWidth: 1,
          padding: [10, 15],
          textStyle: {
            fontSize: 14
          },
          formatter: function(params) {
            let result = `<div style="color:#fff;font-weight:bold;margin-bottom:8px">${params[0].axisValue}</div>`;
            params.forEach(item => {
              let value = item.value;
              let color = item.color;
              let name = item.seriesName;
              result += `<div style="color:${color};font-size:14px;line-height:20px">
                <span style="display:inline-block;width:10px;height:10px;background:${color};margin-right:6px;border-radius:50%"></span>
                ${name}：<span style="float:right;font-weight:bold">${value}个</span>
              </div>`;
            });
            return result;
          }
        },
        legend: {
          data: ['夹杂物', '补丁', '划痕', '其他缺陷'],
          textStyle: {
            color: '#eee',
            fontSize: 14
          },
          icon: 'roundRect',
          itemWidth: 25,
          itemHeight: 14,
          itemGap: 25,
          top: 60,
          padding: [0, 20]
        },
        grid: {
          left: '3%',
          right: '4%',
          top: '20%',
          bottom: '5%',
          containLabel: true
        },
        xAxis: {
          type: 'category',
          boundaryGap: false,
          data: hours,
          axisLabel: {
            color: '#eee',
            fontSize: 12,
            margin: 15
          },
          axisLine: {
            lineStyle: {
              color: 'rgba(255, 255, 255, 0.2)'
            }
          },
          splitLine: {
            show: true,
            lineStyle: {
              color: 'rgba(255, 255, 255, 0.1)',
              type: 'dashed'
            }
          }
        },
        yAxis: {
          type: 'value',
          name: '缺陷数量',
          nameTextStyle: {
            color: '#eee',
            fontSize: 14,
            padding: [0, 0, 0, 50]
          },
          axisLabel: {
            color: '#eee',
            fontSize: 12,
            formatter: '{value}个'
          },
          axisLine: {
            lineStyle: {
              color: 'rgba(255, 255, 255, 0.2)'
            }
          },
          splitLine: {
            lineStyle: {
              color: 'rgba(255, 255, 255, 0.1)',
              type: 'dashed'
            }
          }
        },
        series: [
          {
            name: '夹杂物',
            type: 'line',
            data: this.chartData.inclusion,
            symbol: 'circle',
            symbolSize: 8,
            itemStyle: {
              color: '#FF6B6B',
              borderWidth: 2,
              borderColor: '#fff'
            },
            lineStyle: {
              width: 3,
              shadowColor: 'rgba(255, 107, 107, 0.5)',
              shadowBlur: 10
            },
            emphasis: {
              scale: true,
              focus: 'series'
            },
            smooth: false
          },
          {
            name: '补丁',
            type: 'line',
            data: this.chartData.patch,
            symbol: 'rect',
            symbolSize: 8,
            itemStyle: {
              color: '#4ECDC4',
              borderWidth: 2,
              borderColor: '#fff'
            },
            lineStyle: {
              width: 3,
              shadowColor: 'rgba(78, 205, 196, 0.5)',
              shadowBlur: 10
            },
            emphasis: {
              scale: true,
              focus: 'series'
            },
            smooth: false
          },
          {
            name: '划痕',
            type: 'line',
            data: this.chartData.scratch,
            symbol: 'triangle',
            symbolSize: 10,
            itemStyle: {
              color: '#FFD93D',
              borderWidth: 2,
              borderColor: '#fff'
            },
            lineStyle: {
              width: 3,
              shadowColor: 'rgba(255, 217, 61, 0.5)',
              shadowBlur: 10
            },
            emphasis: {
              scale: true,
              focus: 'series'
            },
            smooth: false
          },
          {
            name: '其他缺陷',
            type: 'line',
            data: this.chartData.otherDefects,
            symbol: 'diamond',
            symbolSize: 9,
            itemStyle: {
              color: '#6C5CE7',
              borderWidth: 2,
              borderColor: '#fff'
            },
            lineStyle: {
              width: 3,
              shadowColor: 'rgba(108, 92, 231, 0.5)',
              shadowBlur: 10
            },
            emphasis: {
              scale: true,
              focus: 'series'
            },
            smooth: false
          }
        ]
      };

      this.chart.setOption(option);
    }
  },
  watch: {
    chartData: {
      deep: true,
      handler(val) {
        this.setOptions(Array.from({length: 24}, (_, i) => `${i}:00`))
      }
    }
  }
}
</script>
