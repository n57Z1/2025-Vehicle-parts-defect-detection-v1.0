<template>
  <el-row :gutter="160" class="panel-group">
    <el-col :xs="24" :sm="12" :lg="8" class="card-panel-col">
      <div class="card-panel" @click="handleSetLineChartData('newVisitis')">
        <div class="card-panel-icon-wrapper icon-people">
          <svg-icon icon-class="cpu" class-name="card-panel-icon" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text">
            检测零件总数
          </div>
          <count-to :start-val="0" :end-val="cpu_percent" :duration="2600" class="card-panel-num" />个
        </div>
      </div>
    </el-col>
    <el-col :xs="24" :sm="12" :lg="8" class="card-panel-col">
      <div class="card-panel" @click="handleSetLineChartData('messages')">
        <div class="card-panel-icon-wrapper icon-message">
          <svg-icon icon-class="memory" class-name="card-panel-icon" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text">
            平均准确率
          </div>
          <count-to :start-val="0" :end-val="memory_percent" :duration="3000" class="card-panel-num" />%
        </div>
      </div>
    </el-col>
    <el-col :xs="24" :sm="12" :lg="8" class="card-panel-col">
      <div class="card-panel" @click="handleSetLineChartData('purchases')">
        <div class="card-panel-icon-wrapper icon-money">
          <svg-icon icon-class="network" class-name="card-panel-icon" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text">
            平均检测速度
          </div>
          <count-to :start-val="0" :end-val="delta" :duration="3200" class="card-panel-num" />ms/个
        </div>
      </div>
    </el-col>

  </el-row>
</template>

<script>
import CountTo from 'vue-count-to'

export default {
  components: {
    CountTo
  },
  data(){
    return {

      cpu_percent:1523,
      memory_percent:98.5,
      delta:50

    }
  },
  mounted(){

      let that = this;

      setInterval(function(){
          
          that.req({
            url: '/util/getComInfo',
            method: 'get'
          }).then(res => {
            console.log(res)

            that.cpu_percent = res.data.cpu_percent;
            that.memory_percent = res.data.memory_percent;
            that.delta = navigator.connection.downlink;

          })

      },10000);

  

  },
  methods: {
    handleSetLineChartData(type) {
      this.$emit('handleSetLineChartData', type)
    }
  }
}
</script>

<style lang="scss" scoped>
.panel-group {
  margin-top: 18px;

  .card-panel-col {
    margin-bottom: 32px;
  }

  .card-panel {
    height: auto;
    cursor: pointer;
    font-size: 12px;
    position: relative;
    overflow: hidden;
    color: #f4efef;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    border-radius: 12px;
    border: 1px solid rgba(255, 255, 255, 0.18);
    box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
    transition: all 0.3s ease;

    &:hover {
      transform: translateY(-5px);
      box-shadow: 0 8px 24px rgba(149, 131, 214, 0.4);

      .card-panel-icon-wrapper {
        color: #fff;
      }

      .icon-people {
        background: #8ecaff;
      }

      .icon-message {
        background: #95a9f3;
      }

      .icon-money {
        background: #a193f2;
      }

      .icon-shopping {
        background: #8e9ff3;
      }
    }

    .icon-people {
      color: #40c9c6;
    }

    .icon-message {
      color: #36a3f7;
    }

    .icon-money {
      color: #f4516c;
    }

    .icon-shopping {
      color: #34bfa3
    }

    .card-panel-icon-wrapper {
      float: left;
      margin: 14px 0 0 14px;
      padding: 16px;
      transition: all 0.38s ease-out;
      border-radius: 6px;
    }

    .card-panel-icon {
      float: left;
      font-size: 48px;
    }

    .card-panel-description {
      float: right;
      font-weight: bold;
      margin: 26px;
      margin-left: 0px;

      @media (max-width: 550px) {
        display: block;
        text-align: center;
        margin: 10px 0;
      }

      .card-panel-text {
        line-height: 18px;
        color: rgb(149, 131, 214);
        font-size: 18px;
        margin-bottom: 12px;
      }

      .card-panel-num {
        font-size: 20px;
        color:#fff;
      }
    }
  }
}

@media (max-width:550px) {
  .card-panel {
    height: auto;
  }

  .card-panel-description {
    display: block;
    text-align: center;
    margin: 10px 0;
  }

  .card-panel-icon-wrapper {
    float: none !important;
    width: 100%;
    height: auto;
    margin: 0 !important;

    .svg-icon {
      display: block;
      margin: 14px auto !important;
      float: none !important;
    }
  }
}
</style>
