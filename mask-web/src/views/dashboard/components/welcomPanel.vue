<template>
  <el-row :gutter="160" class="panel-group">
    <el-col :xs="24" :sm="24" :lg="24" class="card-panel-col">
      <div class="card-panel">
        <div class="card-panel-icon-wrapper icon-mask">
          <svg-icon icon-class="gear" class-name="card-panel-icon" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text" style="display: flex; align-items: center; overflow: hidden;">
            <span class="system-title" style="flex: 1; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">车辆零部件表面缺陷检测系统</span>
          </div>
        </div>
      </div>
    </el-col>
  </el-row>
</template>
<script>
export default {
  props: {
    username: {
      type: String,
      default: ''
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
    height: 108px;
    cursor: pointer;
    font-size: 12px;
    position: relative;
    overflow: hidden;
    color: #666;
    background: rgba(10, 17, 79, 0.25) ;
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.15);
    border-radius: 20px;
    border: 1px solid rgba(255, 255, 255, 0.18);

    &:hover {
      .card-panel-icon-wrapper {
        color: #fff;
      }

      .icon-mask {
        background: #e5e5e5;
      }

      .rectangle{

        width: 1000px;

      }

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
      float: left;
      font-weight: bold;
      margin: 26px;
      margin-left: 60px;

      .card-panel-text {
        color: rgba(0, 0, 0, 0.45);
        font-size: 36px;
        letter-spacing: 10px;

        .system-title {
          background: linear-gradient(to right, #18e6c4, #056bfa);
          -webkit-background-clip: text;
          color: transparent;
          font-weight: 800;
          letter-spacing: 5px;
          text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
          position: relative;
          padding-bottom: 5px;
          
          &::after {
            content: '';
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 2px;
            background: linear-gradient(to right, transparent, #304156, transparent);
            transform: scaleX(0);
            transition: transform 0.3s ease;
          }
          
          &:hover::after {
            transform: scaleX(1);
          }
        }
      }

      .card-panel-num {
        font-size: 20px;
      }
    }

        .username-welcom{

      font-size: 36px;
      float: right;

    }

  }

  @media (max-width: 550px) {
    .card-panel {
      height: auto;
      font-size: 14px;
    }
  }
}
</style>
