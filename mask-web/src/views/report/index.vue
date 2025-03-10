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
      <!-- 数据汇总 -->
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
      <!-- 总结提示 -->
      <div class="analysis-tips" v-if="selectedRows.length">
        <p>{{ analysisTips }}</p>
      </div>
      <div class="enhanced-charts-container">
        <div class="responsive-chart-group">
          <!-- 缺陷饼图 -->
          <div class="chart-enhanced" id="defectPieChart"></div>
          <!-- 批次柱状图 -->
          <div class="chart-enhanced" id="batchBarChart"></div>
          <!-- 平行坐标图 -->
          <div class="chart-enhanced" id="parallelChart"></div>
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
      // 批次数据数组，数据在 created 钩子中生成
      batchData: [],
      selectedRows: [],
      dialogVisible: false,
      charts: {
        pie: null,
        bar: null,
        parallel: null
      },
      // 颜色映射：与扇形图配色一致
      groupColorMap: {
        A: '#D15354',
        B: '#5094D5',
        C: '#F9AD95',
        D: '#ABDBE5'
      }
    }
  },
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
    },
    analysisTips() {
      // 按批次类型分组统计数据
      const groups = {
        A: { totalParts: 0, totalDefects: 0, inclusion: 0, patch: 0, scratch: 0, otherDefects: 0 },
        B: { totalParts: 0, totalDefects: 0, inclusion: 0, patch: 0, scratch: 0, otherDefects: 0 },
        C: { totalParts: 0, totalDefects: 0, inclusion: 0, patch: 0, scratch: 0, otherDefects: 0 },
        D: { totalParts: 0, totalDefects: 0, inclusion: 0, patch: 0, scratch: 0, otherDefects: 0 }
      }
      this.selectedRows.forEach(row => {
        const group = row.batchId.charAt(0)
        if (groups[group]) {
          groups[group].totalParts += row.totalParts
          groups[group].inclusion += row.inclusion
          groups[group].patch += row.patch
          groups[group].scratch += row.scratch
          groups[group].otherDefects += row.otherDefects
        }
      })
      // 计算各组的总缺陷数量、缺陷占比以及最大单项缺陷数
      for (const key in groups) {
        const g = groups[key]
        g.totalDefects = g.inclusion + g.patch + g.scratch + g.otherDefects
        g.defectRatio = g.totalParts ? (g.totalDefects / g.totalParts) : 0
        g.maxDefectTypeCount = Math.max(g.inclusion, g.patch, g.scratch, g.otherDefects)
      }
      // 找出缺陷占比最高的批次类型
      let highestRatioGroup = null
      for (const key in groups) {
        if (!highestRatioGroup || groups[key].defectRatio > groups[highestRatioGroup].defectRatio) {
          highestRatioGroup = key
        }
      }
      // 找出总缺陷数量最多的批次类型
      let highestTotalDefectsGroup = null
      for (const key in groups) {
        if (!highestTotalDefectsGroup || groups[key].totalDefects > groups[highestTotalDefectsGroup].totalDefects) {
          highestTotalDefectsGroup = key
        }
      }
      let tips = ''
      if (highestRatioGroup) {
        tips += `在所选批次中，${highestRatioGroup}批次的缺陷占比最高，最高缺陷占比为 ${(groups[highestRatioGroup].defectRatio * 100).toFixed(2)}%。该批次中，最多的缺陷种类数量为 ${groups[highestRatioGroup].maxDefectTypeCount}。 `
      }
      if (highestTotalDefectsGroup) {
        tips += `同时，${highestTotalDefectsGroup}批次的总缺陷数量最多，为 ${groups[highestTotalDefectsGroup].totalDefects}。请注意该批次零件生产。`
      }
      return tips
    }
  },
  created() {
    this.generateBatchData()
  },
  methods: {
    // 指定数字生成批次数据
    generateBatchData() {
      const groups = [
        { prefix: 'A', count: 25 },
        { prefix: 'B', count: 15 },
        { prefix: 'C', count: 50 },
        { prefix: 'D', count: 55 }
      ]
      const data = []
      groups.forEach(group => {
        for (let i = 1; i <= group.count; i++) {
          // 批次号格式：前缀 + 三位数字
          const batchId = group.prefix + String(i).padStart(3, '0')
          // 使用公式生成数据：总零件数递增，缺陷数据取模得到一些固定数值
          const totalParts = 100 + i * 5 + (group.prefix.charCodeAt(0) - 65) * 10
          const inclusion = (i * 2) % 10      // 夹杂物
          const patch = (i * 3) % 8           // 补丁
          const scratch = (i * 4) % 6         // 划痕
          const otherDefects = (i * 5) % 7    // 其他缺陷
          data.push({ batchId, totalParts, inclusion, patch, scratch, otherDefects })
        }
      })
      this.batchData = data
    },
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
      // 初始化三个图表
      this.charts.pie = echarts.init(document.getElementById('defectPieChart'))
      this.charts.bar = echarts.init(document.getElementById('batchBarChart'))
      this.charts.parallel = echarts.init(document.getElementById('parallelChart'))
      this.updateCharts()
    },
    updateCharts() {
      // 汇总所选批次缺陷数据
      const totalDefects = this.selectedRows.reduce((acc, row) => ({
        inclusion: acc.inclusion + row.inclusion,
        patch: acc.patch + row.patch,
        scratch: acc.scratch + row.scratch,
        otherDefects: acc.otherDefects + row.otherDefects
      }), { inclusion: 0, patch: 0, scratch: 0, otherDefects: 0 })

      // 饼图：缺陷类型分布
      const pieOption = {
        backgroundColor: 'transparent',
        title: {
          text: '缺陷类型分布',
          left: 'center',
          textStyle: { color: '#000000', fontSize: 18 }
        },
        tooltip: { trigger: 'item' },
        color: Object.values(this.groupColorMap),
        series: [{
          type: 'pie',
          radius: ['40%', '70%'],
          data: [
            { value: totalDefects.inclusion, name: '夹杂物' },
            { value: totalDefects.patch, name: '补丁' },
            { value: totalDefects.scratch, name: '划痕' },
            { value: totalDefects.otherDefects, name: '其他缺陷' }
          ],
          label: { color: '#000000' }
        }]
      }

      // 柱状图：各批次缺陷对比
      const barOption = {
        backgroundColor: 'transparent',
        title: {
          text: '批次缺陷对比',
          left: 'center',
          top: '5%',
          textStyle: { color: '#000000', fontSize: 18 }
        },
        tooltip: { trigger: 'axis' },
        legend: {
          data: ['夹杂物', '补丁', '划痕', '其他缺陷'],
          textStyle: { color: '#000000' }
        },
        xAxis: {
          type: 'category',
          data: this.selectedRows.map(row => row.batchId),
          axisLabel: { color: '#000000' }
        },
        yAxis: {
          type: 'value',
          axisLabel: { color: '#000000' }
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

      // 平行坐标图配置
      const parallelData = { A: [], B: [], C: [], D: [] }
      this.selectedRows.forEach(row => {
        const group = row.batchId.charAt(0) // 获取批次类型 A/B/C/D
        if (parallelData[group]) {
          parallelData[group].push([
            row.patch,
            row.inclusion,
            row.scratch,
            row.otherDefects
          ])
        }
      })

      const parallelOption = {
        title: {
          text: '批次缺陷平行坐标图',
          left: 'center',
          textStyle: { color: '#000000', fontSize: 18 }
        },
        tooltip: { trigger: 'item' },
        legend: {
          top: '5%',
          left: 'center',
          data: ['A 批次', 'B 批次', 'C 批次', 'D 批次'],
          textStyle: { color: '#000000' }
        },
        parallel: {
          left: '5%',
          right: '5%',
          bottom: '10%',
          top: '15%',
          parallelAxisDefault: {
            type: 'value',
            nameTextStyle: { color: '#333' },
            axisLine: { lineStyle: { color: '#555' } },
            axisTick: { lineStyle: { color: '#555' } },
            splitLine: { lineStyle: { color: '#ddd' } }
          }
        },
        parallelAxis: [
          { dim: 0, name: '补丁', type: 'value' },
          { dim: 1, name: '夹杂物', type: 'value' },
          { dim: 2, name: '划痕', type: 'value' },
          { dim: 3, name: '其他缺陷', type: 'value' }
        ],
        series: [
          {
            name: 'A 批次',
            type: 'parallel',
            lineStyle: { color: this.groupColorMap.A, width: 2, opacity: 0.7 },
            data: parallelData.A
          },
          {
            name: 'B 批次',
            type: 'parallel',
            lineStyle: { color: this.groupColorMap.B, width: 2, opacity: 0.7 },
            data: parallelData.B
          },
          {
            name: 'C 批次',
            type: 'parallel',
            lineStyle: { color: this.groupColorMap.C, width: 2, opacity: 0.7 },
            data: parallelData.C
          },
          {
            name: 'D 批次',
            type: 'parallel',
            lineStyle: { color: this.groupColorMap.D, width: 2, opacity: 0.7 },
            data: parallelData.D
          }
        ]
      }

      // 更新各个图表
      this.charts.pie.setOption(pieOption)
      this.charts.bar.setOption(barOption)
      this.charts.parallel.setOption(parallelOption)
    },
    cellStyle({ row, rowIndex }) {
      return {
        backgroundColor: rowIndex % 2 === 0 
          ? 'rgba(30, 30, 50, 0.9)'
          : 'rgba(35, 35, 60, 0.9)',
        color: '#ffffff',
        borderBottom: '1px solid rgba(92, 92, 160, 0.2)',
        transition: 'background-color 0.3s',
        textShadow: '0 0 5px rgba(0, 0, 0, 0.5)'
      }
    },
    headerCellStyle() {
      return {
        background: 'linear-gradient(135deg, #6366F1 0%, #8B5CF6 100%)',
        color: '#ffffff',
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
  background: #0a021efa;
  min-height: 100vh;
}
.batch-table {
  margin-bottom: 20px;
  border-radius: 12px;
  overflow: hidden;
  background: #ffffff;
  border: 1px solid rgba(92, 92, 160, 0.2);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  width: 100%;
}
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
  background: #ffffff;
  border-radius: 8px;
  margin-bottom: 20px;
  color: #000000;
  font-size: 16px;
}
/* 新增总结提示样式 */
.analysis-tips {
  padding: 10px 20px;
  background: #f7f7f7;
  border-radius: 8px;
  margin-bottom: 20px;
  color: #333333;
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
  background: rgba(255, 255, 255, 0.8);
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
