<template>
  <div class="container-photo">
    <div class="left-option">
      <el-form>
        <el-form-item label="数据选择">
          <el-select v-model="selectedData" placeholder="请选择数据">
            <el-option v-for="data in dataOptions" :key="data.value" :label="data.label" :value="data.value" />
          </el-select>
        </el-form-item>
        <el-form-item style="margin-left:68px;">
          <el-button type="primary" style="width:200px;" @click="generateReport">生成报告</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="right-present">
      <div v-if="reportData">
        <bar-chart :options="chartOptions.barOption" />
        <pie-chart :options="chartOptions.pieOption" />
      </div>
      <P class="text" v-else>请生成报告以查看数据</P>
    </div>
  </div>
</template>
<script>
import { generateChartOptions } from '@/utils/chartOptions'

export default {
  data: function() {
    return {
      selectedData: '',
      dataOptions: [], // 数据选项
      reportData: null,
      chartOptions: {}
    }
  },
  methods: {
    generateReport() {
      // 模拟从后端获取数据
      const mockData = {
        types: ['类型A', '类型B', '类型C'],
        values: [10, 20, 30]
      };
      // 直接使用模拟数据
      this.reportData = mockData
      this.chartOptions = generateChartOptions(this.reportData)
      
      // 如果需要真实的请求，可以保留以下代码
      /*
      this.req({
        url: '/file/generateReport',
        methods: 'get',
        params: {
          data: this.selectedData
        }
      }).then((res) => {
        this.reportData = res.data
        this.chartOptions = generateChartOptions(this.reportData)
      })
      */
    }
  }
}
</script>
<style>
.container-photo {
  display: flex;
  flex-direction: row;
  height: 80vh;
  margin: 30px;
}

.left-option {
  display: flex;
  flex-direction: column;
  justify-content: center;
  flex: 1;
}

.right-present {
  flex: 2;
  border: dotted #000000;
  border-radius: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
