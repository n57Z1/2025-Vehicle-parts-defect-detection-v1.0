<template>
  <div class="report-container">
    <!-- 批次数据表格 -->
    <div class="batch-table">
      <el-table
        :data="batchData"
        style="width: 100%"
        @selection-change="handleSelectionChange">
        <el-table-column
          type="selection"
          width="55">
        </el-table-column>
        <el-table-column
          prop="batchId"
          label="批次号"
          width="120">
        </el-table-column>
        <el-table-column
          prop="totalParts"
          label="零件总数"
          width="120">
        </el-table-column>
        <el-table-column
          prop="defectA"
          label="缺陷A数量"
          width="120">
        </el-table-column>
        <el-table-column
          prop="defectB"
          label="缺陷B数量"
          width="120">
        </el-table-column>
        <el-table-column
          prop="defectC"
          label="缺陷C数量">
        </el-table-column>
      </el-table>
    </div>

    <!-- 分析按钮 -->
    <div class="analysis-actions">
      <el-button type="primary" @click="generateReport" :disabled="!selectedRows.length">
        生成分析报告
      </el-button>
    </div>

    <!-- 报告弹窗 -->
    <el-dialog
      title="数据分析报告"
      :visible.sync="dialogVisible"
      width="80%"
      class="report-dialog">
      <div class="charts-container">
        <div class="chart" id="defectPieChart"></div>
        <div class="chart" id="batchBarChart"></div>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import * as echarts from 'echarts'

export default {
  data() {
    return {
      // 模拟批次数据
      batchData: [
        { batchId: 'B001', totalParts: 100, defectA: 5, defectB: 3, defectC: 2 },
        { batchId: 'B002', totalParts: 150, defectA: 8, defectB: 4, defectC: 3 },
        { batchId: 'B003', totalParts: 120, defectA: 4, defectB: 6, defectC: 1 },
        { batchId: 'B004', totalParts: 200, defectA: 10, defectB: 7, defectC: 5 },
      ],
      selectedRows: [],
      dialogVisible: false,
      charts: {
        pie: null,
        bar: null
      }
    }
  },
  methods: {
    handleSelectionChange(val) {
      this.selectedRows = val
    },
    generateReport() {
      this.dialogVisible = true
      this.$nextTick(() => {
        this.initCharts()
      })
    },
    initCharts() {
      // 初始化饼图
      this.charts.pie = echarts.init(document.getElementById('defectPieChart'))
      // 初始化柱状图
      this.charts.bar = echarts.init(document.getElementById('batchBarChart'))
      
      this.updateCharts()
    },
    updateCharts() {
      // 处理选中数据
      const totalDefects = {
        'defectA': 0,
        'defectB': 0,
        'defectC': 0
      }
      
      this.selectedRows.forEach(row => {
        totalDefects.defectA += row.defectA
        totalDefects.defectB += row.defectB
        totalDefects.defectC += row.defectC
      })

      // 饼图配置
      const pieOption = {
        backgroundColor: '#1a1a1a',
        title: {
          text: '缺陷类型分布',
          left: 'center',
          textStyle: {
            color: '#ffffff'
          }
        },
        tooltip: {
          trigger: 'item',
          backgroundColor: 'rgba(0,0,0,0.7)',
          borderColor: '#333',
          textStyle: {
            color: '#fff'
          }
        },
        color: ['#8B5CF6', '#6366F1', '#4F46E5'],
        series: [{
          type: 'pie',
          radius: '60%',
          data: [
            { value: totalDefects.defectA, name: '缺陷A' },
            { value: totalDefects.defectB, name: '缺陷B' },
            { value: totalDefects.defectC, name: '缺陷C' }
          ],
          label: {
            color: '#ffffff'
          }
        }]
      }

      // 柱状图配置
      const barOption = {
        backgroundColor: '#1a1a1a',
        title: {
          text: '批次缺陷对比',
          left: 'center',
          textStyle: {
            color: '#ffffff'
          }
        },
        tooltip: {
          trigger: 'axis',
          backgroundColor: 'rgba(0,0,0,0.7)',
          borderColor: '#333',
          textStyle: {
            color: '#fff'
          }
        },
        legend: {
          data: ['缺陷A', '缺陷B', '缺陷C'],
          bottom: 0,
          textStyle: {
            color: '#ffffff'
          }
        },
        xAxis: {
          type: 'category',
          data: this.selectedRows.map(row => row.batchId),
          axisLine: {
            lineStyle: {
              color: '#ffffff'
            }
          },
          axisLabel: {
            color: '#ffffff'
          }
        },
        yAxis: {
          type: 'value',
          axisLine: {
            lineStyle: {
              color: '#ffffff'
            }
          },
          axisLabel: {
            color: '#ffffff'
          },
          splitLine: {
            lineStyle: {
              color: '#333333'
            }
          }
        },
        color: ['#8B5CF6', '#6366F1', '#4F46E5'],
        series: [
          {
            name: '缺陷A',
            type: 'bar',
            data: this.selectedRows.map(row => row.defectA)
          },
          {
            name: '缺陷B',
            type: 'bar',
            data: this.selectedRows.map(row => row.defectB)
          },
          {
            name: '缺陷C',
            type: 'bar',
            data: this.selectedRows.map(row => row.defectC)
          }
        ]
      }

      this.charts.pie.setOption(pieOption)
      this.charts.bar.setOption(barOption)
    }
  }
}
</script>

<style scoped>
.report-container {
  padding: 20px;
  background-color: transparent;
  min-height: 100vh;
}

.batch-table {
  margin-bottom: 20px;
}

.batch-table ::v-deep .el-table {
  background-color: #fefefe;
  color: #ffffff;
  border: none;
}

.batch-table ::v-deep .el-table th {
  background-color: #192137;
  color: #06e899;
  border-bottom: 1px solid #333;
}

.batch-table ::v-deep .el-table td {
  background-color: #1a1a1a;
  color: #ffffff;
  border-bottom: 1px solid #333;
}

.batch-table ::v-deep .el-table--enable-row-hover .el-table__body tr:hover > td {
  background-color: #262626;
}

.analysis-actions {
  margin-bottom: 20px;
  text-align: right;
}

.charts-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}

.chart {
  width: 45%;
  height: 400px;
  margin: 10px;
  background-color: #1a1a1a;
  border-radius: 8px;
}

.report-dialog {
  ::v-deep .el-dialog {
    background-color: #1a1a1a;
    
    .el-dialog__title {
      color: #ffffff;
    }
    
    .el-dialog__body {
      padding: 20px;
      color: #ffffff;
    }
    
    .el-dialog__header {
      background-color: #262626;
      padding: 15px 20px;
    }
  }
}



</style>
