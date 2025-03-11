<template>
  <div class="dashboard-container">
    <WelcomPanel />
    <PanelGroup />
    <el-row class="chart-row" style="background:rgba(0,0,200,0.5);padding:10px 0;margin-bottom:32px;">
      <line-chart :chart-data="lineChartData" />
    </el-row>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import PanelGroup from './components/panelGroup'
import LineChart from './components/lineChart'
import WelcomPanel from './components/welcomPanel'

export default {
  name: 'Dashboard',
  components: {
    PanelGroup,
    LineChart,
    WelcomPanel
  },
  data() {
    return {
      lineChartData: {
        inclusion: new Array(24).fill(0),    // 夹杂物
        patch: new Array(24).fill(0),        // 补丁
        scratch: new Array(24).fill(0),      // 划痕
        otherDefects: new Array(24).fill(0)  // 其他缺陷
      }
    }
  },
  mounted(){
    // 生成随机数据
    const generateBatchData = () => {
      const groups = [
        { prefix: 'A', count: 6 },
        { prefix: 'B', count: 6 },
        { prefix: 'C', count: 6 },
        { prefix: 'D', count: 6 }
      ];
      
      let inclusion = new Array(24).fill(0);
      let patch = new Array(24).fill(0);
      let scratch = new Array(24).fill(0);
      let otherDefects = new Array(24).fill(0);
      
      groups.forEach((group, groupIndex) => {
        for (let i = 1; i <= group.count; i++) {
          const timePoint = groupIndex * 6 + i - 1;
          
          inclusion[timePoint] = Math.floor(Math.random() * 10);
          patch[timePoint] = Math.floor(Math.random() * 8);
          scratch[timePoint] = Math.floor(Math.random() * 6);
          otherDefects[timePoint] = Math.floor(Math.random() * 7);
        }
      });
      
      return { inclusion, patch, scratch, otherDefects };
    };

    // 生成数据并更新图表
    const data = generateBatchData();
    this.lineChartData = {
      inclusion: data.inclusion,
      patch: data.patch,
      scratch: data.scratch,
      otherDefects: data.otherDefects
    };

    // API调用部分
    let that = this;
    this.req({
        url: '/util/getMaskData',
        method: 'get'
    }).then(res => {
      if (res.data && res.data.mask_data) {
        let inclusion = new Array(24).fill(0);
        let patch = new Array(24).fill(0);
        let scratch = new Array(24).fill(0);
        let otherDefects = new Array(24).fill(0);
        
        let mask_data = res.data.mask_data;
        for(let item of mask_data){
          const timePoint = Number(item[2]);
          inclusion[timePoint] = item.inclusion || 0;
          patch[timePoint] = item.patch || 0;
          scratch[timePoint] = item.scratch || 0;
          otherDefects[timePoint] = item.otherDefects || 0;
        }
        
        that.lineChartData = {
          inclusion,
          patch,
          scratch,
          otherDefects
        };
      }
    }).catch(err => {
      console.log('使用模拟数据', err);
    });
  },
  computed: {
    ...mapGetters([
      'name'
    ])
  }
}
</script>

<style lang="scss" scoped> 
.dashboard {
  &-container {
    margin: 30px;
    border-radius: 8px;
    overflow: hidden;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
    color:antiquewhite;
  }
}

// 添加图表行的样式
.chart-row {
  min-height: 500px;  // 设置最小高度，比图表略高
  display: flex;
  align-items: center;
  justify-content: center;
  
  :deep(.el-row) {
    width: 100%;
  }
}
</style>
