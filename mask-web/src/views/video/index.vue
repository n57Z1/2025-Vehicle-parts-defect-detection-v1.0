<template>
  <div class="container-photo">
    <div class="content-wrapper">
      <h2 class="section-title"></h2>
      
      <div class="model-selection">
        <span class="model-label">模型选择</span>
        <el-select v-model="modules" placeholder="请选择合适的模型" class="model-select">
          <el-option label="net_normal(高准确度）" value="net_normal" />
          <el-option label="net_light(高帧数)" value="net_light" />
        </el-select>
        <el-button type="primary" @click="onSubmit" class="detect-btn">立即检测</el-button>
      </div>
      
      <div class="video-upload-area">
        <el-upload
          class="upload-container"
          :action="''"
          :auto-upload="false"
          :show-file-list="false"
          :on-change="handleVideoChange"
          :before-upload="beforeUploadVideo"
        >
          <video
            v-if="videoForm.Video !='' && videoFlag == false"
            :src="videoForm.Video"
            class="video-preview"
            controls="controls"
          >您的浏览器不支持视频播放</video>
          <div v-else-if="videoForm.Video =='' && videoFlag == false" class="upload-icon-container">
            <i class="el-icon-plus upload-icon"></i>
            <div class="upload-text">请保证视频格式正确</div>
            <el-button size="small" type="primary" class="upload-btn">点击上传</el-button>
          </div>
          <el-progress
            v-if="videoFlag == true"
            type="circle"
            :percentage="videoUploadPercent"
            class="progress-circle"
          />
        </el-upload>

        <el-progress 
          v-if="isProcessing"
          :percentage="processPercent"
          :format="format"
          class="processing-progress"
        ></el-progress>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      modules: '',
      uploadUrl: '',
      videoForm: {
        Video: ''
      },
      videoFlag: false,
      videoUploadPercent: 0,
      videoName: '',
      isProcessing: false,
      processPercent: 0
    }
  },
  methods: {
    onSubmit() {
      if (!this.videoForm.Video) {
        this.$message.warning('请先上传视频')
        return
      }

      this.isProcessing = true
      this.processPercent = 0
      
      const loading = this.$loading({
        lock: true,
        text: '检测中...',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      })

      const interval = setInterval(() => {
        this.processPercent += 10
        if (this.processPercent >= 100) {
          clearInterval(interval)
          this.isProcessing = false
          loading.close()
          this.$message.success('视频检测完成')
        }
      }, 300)
    },
    beforeUploadVideo(file) {
      if (
        [
          'video/mp4',
          'video/ogg',
          'video/flv',
          'video/avi',
          'video/wmv',
          'video/rmvb'
        ].indexOf(file.type) == -1
      ) {
        this.$message.error('请上传正确的视频格式')
        return false
      }
      return false;
    },
    handleVideoChange(file) {
      if (file.raw) {
        this.videoFlag = true;
        this.videoUploadPercent = 0;
        
        const interval = setInterval(() => {
          this.videoUploadPercent += 10;
          if (this.videoUploadPercent >= 100) {
            clearInterval(interval);
            this.handleVideoSuccess(null, file);
          }
        }, 200);
      }
    },
    uploadVideoProcess(event, file, fileList) {
      this.videoFlag = true
      console.log('percentage', file.percentage)
      this.videoUploadPercent = file.percentage.toFixed(0)
    },
    handleVideoSuccess(res, file) {
      this.videoFlag = false
      this.videoName = file.name
      
      this.videoForm.Video = URL.createObjectURL(file.raw)
      this.$message.success('视频上传完成')
    },
    format(percentage) {
      return percentage === 100 ? '完成' : `${percentage}%`
    }
  }
}
</script>

<style>
.container-photo {
  min-height: 100vh;
  width: 100%;
  display: flex;
  justify-content: center;
  padding: 20px;
}

.content-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;  /* 使所有子元素水平居中 */
  width: 100%;
  max-width: 1200px;
}

.section-title {
  color: #ffffff;
  font-size: 28px;
  margin-bottom: 30px;
  font-weight: 500;
  text-align: center;  /* 标题居中 */
  width: 100%;
}

.model-selection {
  display: flex;
  align-items: center;
  justify-content: center;  /* 使模型选择部分内容居中 */
  margin-bottom: 120px;
  width: 100%;
}

.model-label {
  color: #ffffff;
  margin-right: 10px;
  font-size: 16px;
}

.model-select {
  width: 240px;
  margin-right: 15px;
}

.detect-btn {
  background-color: #42a5f5;
  border-color: #42a5f5;
}

.video-upload-area {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.upload-container {
  background-color: rgba(255, 255, 255, 0.05);
  border: 2px dashed rgba(66, 165, 245, 0.6);
  border-radius: 12px;
  width: 100%;
  max-width: 800px;
  height: 450px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
}

.upload-container:hover {
  border-color: #42a5f5;
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(66, 165, 245, 0.2);
}

.upload-icon {
  font-size: 60px;
  color: #42a5f5;
  margin-bottom: 20px;
  transition: transform 0.3s ease;
}

.upload-container:hover .upload-icon {
  transform: scale(1.1);
}

.upload-text {
  color: #e0e0e0;
  margin-bottom: 25px;
  font-size: 16px;
}

.upload-btn {
  background-color: #42a5f5;
  border-color: #42a5f5;
  padding: 12px 24px;
  font-size: 16px;
  border-radius: 25px;
  transition: all 0.3s ease;
}

.upload-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(66, 165, 245, 0.3);
}

.progress-circle {
  margin: 20px 0;
}

.processing-progress {
  margin-top: 30px;
  width: 80%;
}

.video-preview {
  max-width: 100%;
  max-height: 400px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}
</style>