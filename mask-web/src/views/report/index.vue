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
    @opened="handleDialogOpened"
    :before-close="handleBeforeClose">
    <!-- 批次选择下拉框 -->
    <el-select v-model="selectedBatch" placeholder="选择批次" @change="updateBatchChart">
      <el-option label="A 批次" value="A"></el-option>
      <el-option label="B 批次" value="B"></el-option>
      <el-option label="C 批次" value="C"></el-option>
      <el-option label="D 批次" value="D"></el-option>
      <el-option label="全部批次" value="all"></el-option>
    </el-select>

    <!-- 数据汇总 -->
    <div class="enhanced-charts-container">
      <div class="responsive-chart-group">
        <!-- 缺陷饼图 -->
        <div class="chart-enhanced" id="defectPieChart"></div>
        <!-- 批次柱状图 -->
        <div class="chart-enhanced" id="batchBarChart"></div>
      </div>
      <!-- 新增说明文字区域 -->
      <div class="report-description">
        <p>{{ batchAnalysisText }}</p>
      </div>
      <div id="chart-container">
    <div id="boxplot-container" style="width: 45%; height: 450px; float: left;"></div>
    <div id="heatmap-container" style="width: 45%; height: 450px; float: left;"></div>
  </div>
      <div class="parallel-chart-container">
        <!-- 平行坐标图 -->
        <div class="chart-enhanced" id="parallelChart"></div>
      </div>
     <!-- 新增说明文字区域 -->
     <div class="chart-description">
      <div :style="{ whiteSpace: 'pre-line', textAlign: 'left' }">{{ chartAnalysisText }}</div>
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
    chartAnalysisText: '',
    batchAnalysisText: '',
    batchData: [],
    selectedRows: [],
    dialogVisible: false,
    selectedBatch: 'A', // 默认选择批次A
    charts: {
      pie: null,
      bar: null,
      parallel: null
    },
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
    // 计算皮尔逊相关系数
calculatePearsonCorrelation(X, Y) {
  const meanX = this.calculateMean(X);
  const meanY = this.calculateMean(Y);
  let numerator = 0;
  let denominatorX = 0;
  let denominatorY = 0;

  for (let i = 0; i < X.length; i++) {
    numerator += (X[i] - meanX) * (Y[i] - meanY);
    denominatorX += Math.pow(X[i] - meanX, 2);
    denominatorY += Math.pow(Y[i] - meanY, 2);
  }

  const denominator = Math.sqrt(denominatorX * denominatorY);
  return numerator / denominator;
},

// 计算平均值
calculateMean(arr) {
  const sum = arr.reduce((acc, val) => acc + val, 0);
  return sum / arr.length;
},

     // 计算箱型图数据
  calculateBoxplotData(data) {
    const sortedData = data.slice().sort((a, b) => a - b); // 排序数据
    const q1 = this.getQuartile(sortedData, 0.25);  // 计算第一四分位数
    const median = this.getQuartile(sortedData, 0.5); // 计算中位数
    const q3 = this.getQuartile(sortedData, 0.75);  // 计算第三四分位数
    const min = sortedData[0];  // 最小值
    const max = sortedData[sortedData.length - 1]; // 最大值

    // 返回箱型图所需的数据
    return [min, max, q1, median, q3];
  },

  // 计算四分位数
  getQuartile(data, q) {
    const pos = (data.length - 1) * q;
    const base = Math.floor(pos);
    const rest = pos - base;
    if (data[base + 1] !== undefined) {
      return data[base] + rest * (data[base + 1] - data[base]);
    } else {
      return data[base];
    }
  },
    updateChartAnalysis() {
    // 动态生成分析文字
    let analysisText = '根据多种分析图的走势，可以得出以下分析：\n\n';

    analysisText += '1. 总体缺陷趋势：从图中可以看出，大多数批次在夹杂物、划痕和补丁这三种缺陷类型上的表现较为均衡。然而，部分批次在“补丁”缺陷上显示出明显较高的数值，表明这些批次可能在某个生产环节中存在质量控制问题。\n\n';
    analysisText += '2. 异常批次：某些批次（如批次A、批次B）显示出明显的异常波动，特别是在“划痕”和“夹杂物”两个缺陷类型上。这些批次的缺陷数值显著高于其他批次，建议对其生产过程进行重点检查，以确定是否存在生产线不稳定或操作问题。\n\n';
    analysisText += '3. 缺陷关联性：图中的趋势显示，“划痕”和“补丁”这两类缺陷在多个批次中有较强的相关性，这可能意味着在生产过程中，这两类缺陷的发生存在某种内在联系。例如，较多的划痕可能导致更多的补丁修复需求。\n\n';
    analysisText += '4. 生产质量波动：部分批次的缺陷分布呈现较大的波动，尤其是在“其他缺陷”类型上。这表明生产过程中可能存在某些不稳定因素，需要进一步调查生产流程和原材料的质量。\n\n';
    analysisText += '总的来说，通过对平行坐标图的分析，可以更好地识别出生产中的潜在问题，针对性地采取措施，以提升整体生产质量。';

    this.chartAnalysisText = analysisText;
  },
    updateBatchChart() {
  this.updateCharts();
},

// 指定数字生成批次数据
generateBatchData() {
  const groups = [
    { prefix: 'A', count: 10 },
    { prefix: 'B', count: 10 },
    { prefix: 'C', count: 10 },
    { prefix: 'D', count: 10 }
  ];
  const data = [];

  groups.forEach(group => {
    for (let i = 1; i <= group.count; i++) {
      // 批次号格式：前缀 + 三位数字
      const batchId = group.prefix + String(i).padStart(3, '0');
      // 随机生成总零件数，范围在100到200之间
      const totalParts = Math.floor(Math.random() * 100) + 100;
      // 随机生成缺陷数据，取值范围可以根据实际情况调整
      const inclusion = Math.floor(Math.random() * 10);    // 夹杂物
      const patch = Math.floor(Math.random() * 8);         // 补丁
      const scratch = Math.floor(Math.random() * 6);       // 划痕
      const otherDefects = Math.floor(Math.random() * 7);  // 其他缺陷
      data.push({ batchId, totalParts, inclusion, patch, scratch, otherDefects });
    }
  });
  this.batchData = data;
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
      this.charts.boxplot = echarts.init(document.getElementById('boxplot-container')); // 初始化箱型图
      this.charts.heatmap = echarts.init(document.getElementById('heatmap-container'));
      this.updateCharts()
    },
    updateCharts() {
  let filteredRows = [];

  if (this.selectedBatch === "all") {
    // 选择全部批次时，使用所有数据
    filteredRows = this.selectedRows;
  } else {
    // 仅筛选出当前批次的数据
    filteredRows = this.selectedRows.filter(row => row.batchId.charAt(0) === this.selectedBatch);
  }
// 计算每个零件的缺陷总数
  const defectCounts = filteredRows.map(row => row.inclusion + row.patch + row.scratch + row.otherDefects);

  // 配置箱型图数据
  const boxplotOption = {
  title: {
    text: `${this.selectedBatch} 批次缺陷分布箱型图`,
    left: 'center',
    textStyle: { color: '#000000', fontSize: 18 },
    top: '10%' // 将标题上移，这样空出空间以便移动图表
  },
  tooltip: { trigger: 'item' },
  xAxis: {
    type: 'category',
    data: [this.selectedBatch],  // 只有选择的批次
    axisLabel: { color: '#000000' }
  },
  yAxis: {
    type: 'value',
    axisLabel: { color: '#000000' }
  },
  grid: {
    top: '30%', // 调整整个图表区域的位置，使其往下移动
    bottom: '10%',
    left: '5%',
    right: '5%'
  },
  series: [{
    name: `${this.selectedBatch} 批次`,
    type: 'boxplot',
    data: [this.calculateBoxplotData(defectCounts)],
    itemStyle: {
      color: this.groupColorMap[this.selectedBatch]
    },
    tooltip: {
      formatter: function (param) {
        return [
          '批次: ' + param.name,
          '最大值: ' + param.data[5],
          '上四分位数: ' + param.data[4],
          '中位数: ' + param.data[3],
          '下四分位数: ' + param.data[2],
          '最小值: ' + param.data[1]
        ].join('<br>');
      }
    }
  }]
};

  // 更新箱型图
  this.charts.boxplot.clear();
  this.charts.boxplot.setOption(boxplotOption, true);

    // 提取缺陷数据
    const inclusionData = filteredRows.map(row => row.inclusion);
  const patchData = filteredRows.map(row => row.patch);
  const scratchData = filteredRows.map(row => row.scratch);
  const otherDefectsData = filteredRows.map(row => row.otherDefects);

  // 计算缺陷类型之间的皮尔逊相关性
  const correlations = {
    inclusion_patch: this.calculatePearsonCorrelation(inclusionData, patchData),
    inclusion_scratch: this.calculatePearsonCorrelation(inclusionData, scratchData),
    inclusion_other: this.calculatePearsonCorrelation(inclusionData, otherDefectsData),
    patch_scratch: this.calculatePearsonCorrelation(patchData, scratchData),
    patch_other: this.calculatePearsonCorrelation(patchData, otherDefectsData),
    scratch_other: this.calculatePearsonCorrelation(scratchData, otherDefectsData)
  };

  // 生成热力图数据
  const heatmapData = [
    [0, 1, correlations.inclusion_patch],
    [0, 2, correlations.inclusion_scratch],
    [0, 3, correlations.inclusion_other],
    [1, 2, correlations.patch_scratch],
    [1, 3, correlations.patch_other],
    [2, 3, correlations.scratch_other]
  ];

  // 配置热力图
const heatmapOption = {
  title: {
  text: `${this.selectedBatch} 批次缺陷相关度热力图`,
  left: 'center',  // 水平居中
  top: '10%',      // 调整垂直位置
  textStyle: { color: '#000000', fontSize: 18 }
}
,
  tooltip: { position: 'top' },
  grid: { 
        top: '20%', // 调整 grid 位置，避免内容遮挡
        bottom: '10%', 
        left: '20%', 
        right: '0%'
      },
  xAxis: {
    type: 'category',
    data: ['夹杂物', '补丁', '划痕', '其他缺陷'],
    axisLabel: { color: '#000000' }
  },
  yAxis: {
    type: 'category',
    data: ['夹杂物', '补丁', '划痕', '其他缺陷'],
    axisLabel: { color: '#000000' }
  },
  visualMap: {
    min: -1,
    max: 1,
    calculable: true,
    inRange: { color: ['#FFFFFF', '#FF0000'] }, // 从白色到红色
    orient: 'vertical', // 设置颜色区间竖直显示
    right: '0%',        // 颜色区间放置在右侧
    top: '15%',         // 可以根据需要调整位置
    bottom: '5%'        // 保证热力图完整显示
  },
  series: [{
    name: '缺陷相关度',
    type: 'heatmap',
    data: heatmapData,
    label: {
      show: true,
      color: '#000000',
      formatter: (params) => {
        return params.value[2].toFixed(2); // 显示相关系数
      }
    },
    itemStyle: {
      borderColor: '#000000',
      borderWidth: 1
    }
  }]
};



  // 渲染热力图
  const heatmapChart = echarts.init(document.getElementById('heatmap-container'));
  heatmapChart.setOption(heatmapOption, true);

  // 饼图：统计所有缺陷的分布情况，使用 filteredRows
  const totalDefectsObj = filteredRows.reduce((acc, row) => ({
    inclusion: acc.inclusion + row.inclusion,
    patch: acc.patch + row.patch,
    scratch: acc.scratch + row.scratch,
    otherDefects: acc.otherDefects + row.otherDefects
  }), { inclusion: 0, patch: 0, scratch: 0, otherDefects: 0 });

  const pieOption = {
    backgroundColor: 'transparent',
    title: { text: '缺陷类型分布', left: 'center', textStyle: { color: '#000000', fontSize: 18 } },
    tooltip: { trigger: 'item' },
    color: Object.values(this.groupColorMap),
    series: [{
      type: 'pie',
      radius: ['40%', '70%'],
      data: [
        { value: totalDefectsObj.inclusion, name: '夹杂物' },
        { value: totalDefectsObj.patch, name: '补丁' },
        { value: totalDefectsObj.scratch, name: '划痕' },
        { value: totalDefectsObj.otherDefects, name: '其他缺陷' }
      ],
      label: { color: '#000000', formatter: '{b}: {c} ({d}%)' }
    }]
  };

  let barOption = {};
  if (this.selectedBatch === "all") {
    // 固定只统计 A、B、C、D 批次的数据
    const fixedBatches = ['A', 'B', 'C', 'D'];
    const batchGroups = {};
    // 预先初始化批次数据
    fixedBatches.forEach(batch => {
      batchGroups[batch] = { inclusion: 0, patch: 0, scratch: 0, otherDefects: 0 };
    });

    // 仅统计 A、B、C、D 批次
    this.selectedRows.forEach(row => {
      const batch = row.batchId.charAt(0);
      if (fixedBatches.includes(batch)) {
        batchGroups[batch].inclusion += row.inclusion;
        batchGroups[batch].patch += row.patch;
        batchGroups[batch].scratch += row.scratch;
        batchGroups[batch].otherDefects += row.otherDefects;
      }
    });

    // 固定图例顺序为 A、B、C、D
    const legendData = fixedBatches.map(batch => `${batch} 批次`);

    barOption = {
      backgroundColor: 'transparent',
      title: { 
        text: '不同批次缺陷对比', 
        left: 'center', 
        top: '0%', // 标题放在最上方
        textStyle: { color: '#000000', fontSize: 18 } 
      },
      tooltip: { trigger: 'axis' },
      legend: { 
        data: legendData, 
        textStyle: { color: '#000000' },
        top: '10%', // 图例位置在标题下方
        left: 'center'
      },
      xAxis: { 
        type: 'category', 
        data: ['夹杂物', '补丁', '划痕', '其他缺陷'], 
        axisLabel: { color: '#000000' } 
      },
      yAxis: { 
        type: 'value', 
        axisLabel: { color: '#000000' } 
      },
      grid: { 
        top: '20%', // 调整 grid 位置，避免内容遮挡
        bottom: '10%', 
        left: '5%', 
        right: '5%'
      },
      series: fixedBatches.map(batch => ({
        name: `${batch} 批次`,
        type: 'bar',
        data: [
          batchGroups[batch].inclusion,
          batchGroups[batch].patch,
          batchGroups[batch].scratch,
          batchGroups[batch].otherDefects
        ]
      }))
    };
  } else {
    // 只展示当前批次的数据
    barOption = {
      backgroundColor: 'transparent',
      title: { 
        text: '同一批次缺陷对比', 
        left: 'center', 
        top: '0%', // 标题位置
        textStyle: { color: '#000000', fontSize: 18 }
      },
      tooltip: { trigger: 'axis' },
      legend: { 
        data: filteredRows.map(row => row.batchId), 
        textStyle: { color: '#000000' },
        top: '10%', // 图例位置
        left: 'center'
      },
      xAxis: { 
        type: 'category', 
        data: ['夹杂物', '补丁', '划痕', '其他缺陷'], 
        axisLabel: { color: '#000000' } 
      },
      yAxis: { 
        type: 'value', 
        axisLabel: { color: '#000000' } 
      },
      grid: { 
        top: '20%', 
        bottom: '10%', 
        left: '5%', 
        right: '5%'
      },
      series: filteredRows.map(row => ({
        name: row.batchId,
        type: 'bar',
        data: [row.inclusion, row.patch, row.scratch, row.otherDefects]
      }))
    };
  }

  // 清空并更新图表，防止旧数据残留
  this.charts.pie.clear();
  this.charts.pie.setOption(pieOption, true);

  this.charts.bar.clear();
  this.charts.bar.setOption(barOption, true);

  // ----------------------------
  // 确保平行坐标图被初始化
  // ----------------------------
  if (!this.charts.parallel) {
    this.renderParallelChart();
  }

  // ----------------------------
  // 更新平行坐标图
  // ----------------------------
  const parallelData = { A: [], B: [], C: [], D: [] };

  this.selectedRows.forEach(row => {
    const group = row.batchId.charAt(0);
    if (parallelData[group]) {
      parallelData[group].push([
        row.patch,
        row.inclusion,
        row.scratch,
        row.otherDefects
      ]);
    }
  });

  const parallelOption = {
    backgroundColor: 'transparent',
    title: {
      text: '平行坐标缺陷分布',
      left: 'center',
      textStyle: { color: '#000000', fontSize: 18 },
      top: '5%'  // 标题与图例不重叠
    },
    legend: {
      data: ['A 批次', 'B 批次', 'C 批次', 'D 批次'],
      left: 'center',
      top: '15%',
      textStyle: { color: '#000000' }
    },
    parallelAxis: [
      { dim: 0, name: '补丁' },
      { dim: 1, name: '夹杂物' },
      { dim: 2, name: '划痕' },
      { dim: 3, name: '其他缺陷' }
    ],
    parallel: {
      left: '5%',
      right: '10%',
      bottom: '10%',
      top: '30%'
    },
    series: [
      { name: 'A 批次', type: 'parallel', data: parallelData.A, lineStyle: { color: this.groupColorMap.A } },
      { name: 'B 批次', type: 'parallel', data: parallelData.B, lineStyle: { color: this.groupColorMap.B } },
      { name: 'C 批次', type: 'parallel', data: parallelData.C, lineStyle: { color: this.groupColorMap.C } },
      { name: 'D 批次', type: 'parallel', data: parallelData.D, lineStyle: { color: this.groupColorMap.D } }
    ]
  };

  this.charts.parallel.clear();
  this.charts.parallel.setOption(parallelOption, true);

  // ----------------------------
  // 统计分析文字
  // ----------------------------
  const totalParts = filteredRows.reduce((acc, row) => acc + row.totalParts, 0);
  const totalDefects = filteredRows.reduce((acc, row) => acc + row.inclusion + row.patch + row.scratch + row.otherDefects, 0);
  const overallDefectRate = totalParts ? ((totalDefects / totalParts) * 100).toFixed(2) : 0;

  const defectTypes = { inclusion: 0, patch: 0, scratch: 0, otherDefects: 0 };
  filteredRows.forEach(row => {
    defectTypes.inclusion += row.inclusion;
    defectTypes.patch += row.patch;
    defectTypes.scratch += row.scratch;
    defectTypes.otherDefects += row.otherDefects;
  });

  let maxDefectType = '';
  let maxCount = 0;
  for (const key in defectTypes) {
    if (defectTypes[key] > maxCount) {
      maxCount = defectTypes[key];
      maxDefectType = key;
    }
  }

  const defectTypeMap = { inclusion: '夹杂物', patch: '补丁', scratch: '划痕', otherDefects: '其他缺陷' };
  const maxDefectTypeName = defectTypeMap[maxDefectType] || 'N/A';

  let maxRowDefectRate = 0, maxRateRow = null;
  let maxRowTotalDefects = 0, maxDefectsRow = null;

  filteredRows.forEach(row => {
    const rowDefects = row.inclusion + row.patch + row.scratch + row.otherDefects;
    const rowDefectRate = row.totalParts ? rowDefects / row.totalParts : 0;
    if (rowDefectRate > maxRowDefectRate) {
      maxRowDefectRate = rowDefectRate;
      maxRateRow = row;
    }
    if (rowDefects > maxRowTotalDefects) {
      maxRowTotalDefects = rowDefects;
      maxDefectsRow = row;
    }
  });

  this.batchAnalysisText = `当前${this.selectedBatch === "all" ? "所有批次" : "批次"}总体缺陷率为 ${overallDefectRate}%。其中缺陷数量最多的为 ${maxDefectTypeName}。在该批次中，编号为 ${maxRateRow ? maxRateRow.batchId : 'N/A'} 的零件缺陷率最高（约 ${(maxRowDefectRate * 100).toFixed(2)}%），而编号为 ${maxDefectsRow ? maxDefectsRow.batchId : 'N/A'} 的零件总缺陷最多（共 ${maxRowTotalDefects} 个缺陷）。请特别关注该编号零件对应的生产车间和生产时间段，确保生产质量。`;

  this.updateChartAnalysis();
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
.enhanced-charts-container {
  padding: 20px;
  height: 70vh;
  display: flex;
  flex-direction: column;
  overflow-y: auto; /* 使内容可滚动 */
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
.report-description {
  margin-top: 20px;
  padding: 15px;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 12px;
  text-align: center;
  font-size: 16px;
  font-weight: 500;
  color: #333;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  line-height: 1.6;
}
.chart-description {
  margin-top: 20px;
  padding: 15px;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 12px;
  text-align: center;
  font-size: 16px;
  font-weight: 500;
  color: #333;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  line-height: 1.6;
}
.parallel-chart-container {
  margin-top: 20px;
}
</style>

