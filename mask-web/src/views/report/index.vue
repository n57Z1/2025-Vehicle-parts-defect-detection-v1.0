<template>
  <div class="report-container">
    <!-- 批次数据表格 -->
    <div class="batch-table">
      <el-table
        :data="batchData"
        style="width: 100%"
        @selection-change="handleSelectionChange"
        :cell-style="cellStyle"
        :header-cell-style="headerCellStyle"
        border>
        <el-table-column type="selection" width="55" align="center"></el-table-column>
        <el-table-column prop="batchId" label="批次号" align="center"></el-table-column>
        <el-table-column prop="totalParts" label="零件总数" align="center"></el-table-column>
        <el-table-column label="缺陷统计" header-align="center">
          <el-table-column prop="inclusion" label="夹杂物" align="center"></el-table-column>
          <el-table-column prop="patch" label="补丁" align="center"></el-table-column>
          <el-table-column prop="scratch" label="划痕" align="center"></el-table-column>
          <el-table-column prop="otherDefects" label="其他缺陷" align="center"></el-table-column>
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
      width="90%"
      top="5vh"
      custom-class="large-report-dialog"
      @opened="handleDialogOpened">
      <!-- 扩充的报告内容：数据汇总 -->
      <div class="report-summary" v-if="selectedRows.length">
        <p>选中批次数：{{ reportSummary.count }}</p>
        <p>总零件数：{{ reportSummary.totalParts }}</p>
        <p>
          缺陷汇总： 
          夹杂物 {{ reportSummary.totalDefects.inclusion }}，
          补丁 {{ reportSummary.totalDefects.patch }}，
          划痕 {{ reportSummary.totalDefects.scratch }}，
          其他缺陷 {{ reportSummary.totalDefects.otherDefects }}
        </p>
      </div>
      <div class="enhanced-charts-container">
        <div class="responsive-chart-group">
          <div class="chart-enhanced" id="defectPieChart"></div>
          <div class="chart-enhanced" id="batchBarChart"></div>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import * as echarts from 'echarts'

export default {
  data() {
  return {
    batchData: [
      { batchId: 'B001', totalParts: 100, inclusion: 5, patch: 3, scratch: 2, otherDefects: 1 },
      { batchId: 'B002', totalParts: 150, inclusion: 8, patch: 4, scratch: 3, otherDefects: 2 },
      { batchId: 'B003', totalParts: 120, inclusion: 4, patch: 6, scratch: 1, otherDefects: 3 },
      { batchId: 'B004', totalParts: 200, inclusion: 10, patch: 7, scratch: 5, otherDefects: 4 },
      { batchId: 'B005', totalParts: 110, inclusion: 6, patch: 2, scratch: 1, otherDefects: 0 },
      { batchId: 'B006', totalParts: 180, inclusion: 9, patch: 5, scratch: 4, otherDefects: 3 },
      { batchId: 'B007', totalParts: 130, inclusion: 7, patch: 3, scratch: 2, otherDefects: 1 },
      { batchId: 'B008', totalParts: 160, inclusion: 8, patch: 6, scratch: 3, otherDefects: 2 },
      { batchId: 'B009', totalParts: 140, inclusion: 5, patch: 4, scratch: 2, otherDefects: 3 },
      { batchId: 'B010', totalParts: 190, inclusion: 11, patch: 8, scratch: 6, otherDefects: 4 },
      { batchId: 'B011', totalParts: 210, inclusion: 13, patch: 9, scratch: 7, otherDefects: 5 },
      { batchId: 'B012', totalParts: 150, inclusion: 6, patch: 4, scratch: 3, otherDefects: 2 },
      { batchId: 'B013', totalParts: 175, inclusion: 9, patch: 5, scratch: 4, otherDefects: 3 },
      { batchId: 'B014', totalParts: 160, inclusion: 7, patch: 6, scratch: 2, otherDefects: 3 },
      { batchId: 'B015', totalParts: 200, inclusion: 12, patch: 8, scratch: 5, otherDefects: 4 }
    ],
    selectedRows: [],
    dialogVisible: false,
    charts: {
      pie: null,
      bar: null
    }
  }
}
,
  computed: {
    reportSummary() {
      const totalParts = this.selectedRows.reduce((sum, row) => sum + row.totalParts, 0)
      const totalDefects = this.selectedRows.reduce((acc, row) => ({
        inclusion: acc.inclusion + row.inclusion,
        patch: acc.patch + row.patch,
        scratch: acc.scratch + row.scratch,
        otherDefects: acc.otherDefects + row.otherDefects
      }), { inclusion: 0, patch: 0, scratch: 0, otherDefects: 0 })
      return { count: this.selectedRows.length, totalParts, totalDefects }
    }
  },
  methods: {
    handleSelectionChange(val) {
      this.selectedRows = val
    },
    generateReport() {
      this.dialogVisible = true
    },
    handleDialogOpened() {
      this.$nextTick(() => {
        this.initCharts()
      })
    },
    initCharts() {
      this.charts.pie = echarts.init(document.getElementById('defectPieChart'))
      this.charts.bar = echarts.init(document.getElementById('batchBarChart'))
      this.updateCharts()
    },
    updateCharts() {
      const totalDefects = this.selectedRows.reduce((acc, row) => ({
        inclusion: acc.inclusion + row.inclusion,
        patch: acc.patch + row.patch,
        scratch: acc.scratch + row.scratch,
        otherDefects: acc.otherDefects + row.otherDefects
      }), { inclusion: 0, patch: 0, scratch: 0, otherDefects: 0 })

      const pieOption = {
        backgroundColor: 'transparent',
        title: {
          text: '缺陷类型分布',
          left: 'center',
          textStyle: { color: '#000000', fontSize: 18 } // 文字改为黑色
        },
        tooltip: { trigger: 'item' },
        color: ['#D15354', '#5094D5', '#F9AD95', '#ABDBE5'], // 设置新的扇形图配色
        series: [{
          type: 'pie',
          radius: ['40%', '70%'],
          data: [
            { value: totalDefects.inclusion, name: '夹杂物' },
            { value: totalDefects.patch, name: '补丁' },
            { value: totalDefects.scratch, name: '划痕' },
            { value: totalDefects.otherDefects, name: '其他缺陷' }
          ],
          label: { color: '#000000' } // 扇形图文字也改为黑色
        }]
      }

      const barOption = {
        backgroundColor: 'transparent',
        title: {
          text: '批次缺陷对比',
          left: 'center',
          top: '5%',
          textStyle: { color: '#000000', fontSize: 18 } // 文字改为黑色
        },
        tooltip: { trigger: 'axis' },
        legend: {
          data: ['夹杂物', '补丁', '划痕', '其他缺陷'],
          textStyle: { color: '#000000' } // 文字改为黑色
        },
        xAxis: {
          type: 'category',
          data: this.selectedRows.map(row => row.batchId),
          axisLabel: { color: '#000000' } // x轴标签改为黑色
        },
        yAxis: {
          type: 'value',
          axisLabel: { color: '#000000' } // y轴标签改为黑色
        },
        grid: {
          top: '15%',
          bottom: '10%',
          left: '5%',
          right: '5%'
        },
        series: [
          { name: '夹杂物', type: 'bar', data: this.selectedRows.map(row => row.inclusion) },
          { name: '补丁', type: 'bar', data: this.selectedRows.map(row => row.patch) },
          { name: '划痕', type: 'bar', data: this.selectedRows.map(row => row.scratch) },
          { name: '其他缺陷', type: 'bar', data: this.selectedRows.map(row => row.otherDefects) }
        ]
      }

      this.charts.pie.setOption(pieOption)
      this.charts.bar.setOption(barOption)
    },
    cellStyle({ row, rowIndex }) {
  return {
    backgroundColor: rowIndex % 2 === 0 
      ? 'rgba(30, 30, 50, 0.9)' // 更深的背景色，增加对比度
      : 'rgba(35, 35, 60, 0.9)', // 更深的背景色，增加对比度
    color: '#ffffff', // 白色文字
    borderBottom: '1px solid rgba(92, 92, 160, 0.2)',
    transition: 'background-color 0.3s',
    textShadow: '0 0 5px rgba(0, 0, 0, 0.5)' // 文字阴影，增强可读性
  }
}

,


headerCellStyle() {
  return {
    background: 'linear-gradient(135deg, #6366F1 0%, #8B5CF6 100%)', // 使用按钮的渐变背景
    color: '#ffffff', // 文字颜色为白色
    fontWeight: '600',
    borderBottom: 'none'
  }
}

  }
}
</script>

<style scoped>
.report-container {
  padding: 20px;
  background: #0a021efa; /* 恢复为黑灰色背景 */
  min-height: 100vh;
}
.batch-table {
  margin-bottom: 20px;
  border-radius: 12px;
  overflow: hidden;
  background: #ffffff; /* 表格背景白色 */
  border: 1px solid rgba(92, 92, 160, 0.2);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1); /* 渐弱的阴影效果 */
  width: 100%;
}
/* 固定表格布局，保证各列均分 */
.batch-table ::v-deep .el-table {
  table-layout: fixed;
  width: 100%;
  background: transparent;
}
.batch-table ::v-deep .el-table__header-wrapper,
.batch-table ::v-deep .el-table__body-wrapper {
  overflow: visible;
}
.batch-table ::v-deep .el-table__row:hover > td {
  background-color: rgba(92, 92, 160, 0.3) !important;
  color: #ffffff !important;
}
.batch-table ::v-deep .el-table__cell {
  padding: 12px 0;
  text-overflow: ellipsis;
}
.analysis-actions {
  margin-bottom: 20px;
  text-align: right;
}
.analysis-actions .el-button {
  background: linear-gradient(135deg, #6366F1 0%, #8B5CF6 100%);
  border: none;
  border-radius: 8px;
  padding: 12px 28px;
  font-weight: 600;
  box-shadow: 0 4px 15px rgba(99, 102, 241, 0.3);
}
.report-summary {
  padding: 10px 20px;
  background: #ffffff; /* 背景颜色白色 */
  border-radius: 8px;
  margin-bottom: 20px;
  color: #000000; /* 文字颜色黑色 */
  font-size: 16px;
}
.enhanced-charts-container {
  padding: 20px;
  height: 70vh;
  display: flex;
  flex-direction: column;
}
.responsive-chart-group {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(500px, 1fr));
  gap: 30px;
  flex-grow: 1;
}
.chart-enhanced {
  width: 100%;
  height: 100%;
  min-height: 400px;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.8); /* 背景颜色改为白色 */
}
@media (max-width: 768px) {
  .large-report-dialog {
    width: 95% !important;
    top: 1vh !important;
  }
  .responsive-chart-group {
    grid-template-columns: 1fr;
  }
  .chart-enhanced {
    min-height: 300px;
  }
}
</style>
