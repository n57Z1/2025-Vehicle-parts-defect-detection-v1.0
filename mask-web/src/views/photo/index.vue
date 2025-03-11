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
      
      <div class="photo-upload-area">
        <div v-if="photoUrls.length === 0" class="upload-container">
          <el-upload
            class="upload-demo"
            drag
            action=""
            :http-request="handleManualUpload"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :before-upload="handleBefore"
            multiple
          >
            <i class="el-icon-upload upload-icon" />
            <div class="upload-text"> 请将图片拖到此处，
              <br>
              或<em>点击上传</em></div>
            <div slot="tip" class="el-upload__tip">请上传jpg / png格式的图片</div>
          </el-upload>
        </div>
        
        <div v-if="photoUrls.length > 0" class="detection-results">
          <div v-for="(url, index) in photoUrls" :key="index" class="photo-preview">
            <img :src="url" alt="" srcset="">
          </div>
          <el-card class="result-card">
            <div slot="header" class="result-header">
              <span>检测结果报告</span>
            </div>
            <div class="result-content">
              <el-descriptions :column="1" border>
                <el-descriptions-item label="检测时间">{{ detectionTime }}</el-descriptions-item>
                <el-descriptions-item label="使用模型">{{ modules }}</el-descriptions-item>
                <el-descriptions-item label="检测对象数量">{{ detectionResults.totalObjects || 0 }}</el-descriptions-item>
                <el-descriptions-item label="置信度">{{ detectionResults.confidence || '0%' }}</el-descriptions-item>
              </el-descriptions>
              
              <div class="statistics">
                <el-table :data="detectionResults.details" style="width: 100%">
                  <el-table-column prop="category" label="类别"></el-table-column>
                  <el-table-column prop="count" label="数量"></el-table-column>
                  <el-table-column prop="percentage" label="占比"></el-table-column>
                </el-table>
              </div>
            </div>
          </el-card>
          <el-button type="primary" class="back-btn" @click="onBack">重新上传</el-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      modules: '',
      photoUrls: [],
      imageNames: [],
      isLoading: false,
      detectionTime: '',
      detectionResults: {
        totalObjects: 0,
        confidence: '0%',
        details: []
      }
    }
  },
  methods: {
    onSubmit() {
      const that = this
      this.isLoading = true
      
      // 模拟API请求延迟
      setTimeout(() => {
        // 模拟检测结果
        this.photoUrls = ['/api/placeholder/400/300', '/api/placeholder/400/300']
        this.isLoading = false
        
        // 显示成功提示
        this.$message({
          message: '检测完成!',
          type: 'success'
        });
      }, 2000)
    },
    
    onBack() {
      this.photoUrls = []
      this.imageNames = []
    },
    
    handleBefore(file) {
      // 检查文件类型和大小
      const isImageType = file.type === 'image/jpeg' || file.type === 'image/png'

      if (!isImageType) {
        this.$message.error('只能上传 JPG 或 PNG 格式的图片!')
        return false
      }
      
      return true
    },
    
    handleManualUpload(options) {
      const file = options.file;
      
      // 显示加载中
      const loading = this.$loading({
        lock: true,
        text: '处理中...',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      });
      
      // 三秒后显示完成
      setTimeout(() => {
        // 关闭加载中
        loading.close();
        
        // 创建预览URL
        const imageUrl = URL.createObjectURL(file);
        
        // 添加图片到预览列表
        this.photoUrls.push(imageUrl);
        this.imageNames.push(file.name);
        this.photoUrl = imageUrl;
        
        // 显示成功消息
        this.$message({
          message: '图片处理完成!',
          type: 'success'
        });
      }, 3000);
    },
    
    handleRemove(file, fileList) {
      const index = this.photoUrls.indexOf(file.url);
      if (index !== -1) {
        this.photoUrls.splice(index, 1);
        this.imageNames.splice(index, 1);
      }
    },
    
    handlePreview(file) {
      // 显示预览功能
      console.log('预览文件:', file); // 添加调试信息
      const imageUrl = URL.createObjectURL(file.raw);
      console.log('生成的图片URL:', imageUrl); // 添加调试信息
      const imgWindow = window.open('', '_blank');
      imgWindow.document.write(`<img src="${imageUrl}" alt="Image Preview" style="width: 100%; height: auto;">`);
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
  align-items: center;
  width: 100%;
  max-width: 1200px;
}

.section-title {
  color: #ffffff;
  font-size: 28px;
  margin-bottom: 30px;
  font-weight: 500;
  text-align: center;
  width: 100%;
}

.model-selection {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 50px;
  width: 100%;
}

.model-label {
  margin-right: 10px;
  font-size: 16px;
}

.model-select {
  width: 240px;
  margin-right: 15px;
}

.detect-btn {
  background-color: #409EFF;
  border-color: #409EFF;
  width: 150px;
}

.photo-upload-area {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.upload-container {
  background-color: #ffffff;
  border: 2px dashed #409EFF;
  border-radius: 8px;
  width: 100%;
  max-width: 800px;
  padding: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.upload-icon {
  font-size: 50px;
  color: #c0c4cc;
  margin-bottom: 20px;
}

.upload-text {
  color: #909399;
  margin-bottom: 20px;
}

.detection-results {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.photo-preview {
  margin-bottom: 20px;
  max-width: 800px;
}

.photo-preview img {
  max-width: 100%;
  border-radius: 8px;
}

.result-card {
  width: 90%;
  margin-top: 20px;
  max-width: 800px;
}

.result-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.result-content {
  margin-top: 20px;
}

.statistics {
  margin-top: 20px;
}

.back-btn {
  margin-top: 30px;
  width: 200px;
  background-color: #409EFF;
  border-color: #409EFF;
}

/* 媒体查询以适应手机布局 */
@media (max-width: 768px) {
  .model-selection {
    flex-direction: column;
    align-items: center;
  }
  
  .model-label {
    margin-bottom: 10px;
  }
  
  .model-select {
    margin-right: 0;
    margin-bottom: 10px;
    width: 100%;
  }
  
  .detect-btn {
    width: 100%;
  }
  
  .result-card {
    width: 100%;
  }
}
</style>
