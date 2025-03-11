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
        <div class="upload-container">
          <el-upload
            class="upload-demo"
            drag
            action=""
            :http-request="handleManualUpload"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :before-upload="handleBefore"
            :show-file-list="true"
            multiple
            :file-list="fileList"
          >
            <div class="upload-content">
              <i class="el-icon-upload upload-icon" />
              <div class="upload-text">
                将文件拖到此处，或
                <el-button type="text" @click="openFileDialog">选择文件/文件夹</el-button>
              </div>
              <div class="upload-tip">支持jpg/png格式图片，可多选或上传整个文件夹</div>
            </div>
          </el-upload>
          
          <input
            type="file"
            ref="folderInput"
            style="display: none"
            webkitdirectory
            multiple
            @change="handleFolderSelect"
          >
        </div>
      </div>

      <!-- 浮窗形式的检测报告 -->
      <el-dialog
        title="检测结果报告"
        :visible.sync="showReport"
        width="50%"
        class="detection-report"
      >
        <div class="report-content">
          <div class="report-summary">
            <el-row :gutter="20">
              <el-col :span="6">
                <div class="stat-card">
                  <div class="stat-value">{{ detectionResults.totalObjects }}</div>
                  <div class="stat-label">检测对象总数</div>
                </div>
              </el-col>
              <el-col :span="6">
                <div class="stat-card">
                  <div class="stat-value">{{ detectionResults.confidence }}</div>
                  <div class="stat-label">平均置信度</div>
                </div>
              </el-col>
              <el-col :span="6">
                <div class="stat-card">
                  <div class="stat-value">{{ fileList.length }}</div>
                  <div class="stat-label">处理图片数</div>
                </div>
              </el-col>
              <el-col :span="6">
                <div class="stat-card">
                  <div class="stat-value">{{ detectionResults.processTime }}s</div>
                  <div class="stat-label">处理用时</div>
                </div>
              </el-col>
            </el-row>
          </div>

          <div class="report-details">
            <el-tabs v-model="activeTab">
              <el-tab-pane label="类别统计" name="categories">
                <el-table :data="detectionResults.details" style="width: 100%">
                  <el-table-column prop="category" label="类别"></el-table-column>
                  <el-table-column prop="count" label="数量"></el-table-column>
                  <el-table-column prop="percentage" label="占比">
                    <template slot-scope="scope">
                      <el-progress :percentage="scope.row.percentage" :format="percentageFormat"></el-progress>
                    </template>
                  </el-table-column>
                </el-table>
              </el-tab-pane>
              <el-tab-pane label="图片预览" name="preview">
                <el-carousel :interval="4000" type="card" height="300px">
                  <el-carousel-item v-for="(url, index) in photoUrls" :key="index">
                    <img :src="url" class="preview-image">
                  </el-carousel-item>
                </el-carousel>
              </el-tab-pane>
            </el-tabs>
          </div>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      modules: '',
      photoUrls: [],
      fileList: [],
      showReport: false,
      activeTab: 'categories',
      detectionResults: {
        totalObjects: 156,
        confidence: '92.5%',
        processTime: 3.2,
        details: [
          { category: '表面裂纹', count: 45, percentage: 28.8 },
          { category: '材料变形', count: 32, percentage: 20.5 },
          { category: '腐蚀损坏', count: 28, percentage: 17.9 },
          { category: '表面划痕', count: 21, percentage: 13.5 },
          { category: '结构断裂', count: 18, percentage: 11.5 },
          { category: '其他缺陷', count: 12, percentage: 7.8 }
        ]
      }
    }
  },
  methods: {
    openFileDialog() {
      this.$refs.folderInput.click()
    },
    
    handleFolderSelect(event) {
      const files = Array.from(event.target.files)
      this.processFiles(files)
    },
    
    processFiles(files) {
      const imageFiles = files.filter(file => 
        file.type === 'image/jpeg' || file.type === 'image/png'
      )
      
      imageFiles.forEach(file => {
        this.fileList.push({
          name: file.name,
          url: URL.createObjectURL(file)
        })
        this.photoUrls.push(URL.createObjectURL(file))
      })
    },
    
    handleManualUpload({ file }) {
      this.processFiles([file])
    },
    
    onSubmit() {
      if (this.fileList.length === 0) {
        this.$message.warning('请先上传图片')
        return
      }

      const loading = this.$loading({
        lock: true,
        text: '正在处理图片...',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      })

      setTimeout(() => {
        loading.close()
        this.showReport = true
        this.$message.success('检测完成！')
      }, 2000)
    },
    
    percentageFormat(val) {
      return `${val}%`
    },
    
    handleRemove(file) {
      const index = this.fileList.findIndex(item => item.url === file.url)
      if (index !== -1) {
        this.fileList.splice(index, 1)
        this.photoUrls.splice(index, 1)
      }
    },
    
    handlePreview(file) {
      window.open(file.url)
    },
    
    handleBefore(file) {
      // 检查文件类型和大小
      const isImageType = file.type === 'image/jpeg' || file.type === 'image/png'

      if (!isImageType) {
        this.$message.error('只能上传 JPG 或 PNG 格式的图片!')
        return false
      }
      
      return true
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
  background-color: rgba(255, 255, 255, 0.05);
  border: 2px dashed rgba(64, 158, 255, 0.6);
  border-radius: 12px;
  width: 100%;
  max-width: 800px;
  padding: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
}

.upload-container:hover {
  border-color: #409EFF;
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(64, 158, 255, 0.2);
}

.upload-icon {
  font-size: 60px;
  color: #409EFF;
  margin-bottom: 25px;
  transition: transform 0.3s ease;
}

.upload-container:hover .upload-icon {
  transform: scale(1.1);
}

.upload-text {
  color: #e0e0e0;
  margin-bottom: 25px;
  font-size: 16px;
  line-height: 1.6;
}

.upload-text em {
  color: #409EFF;
  font-style: normal;
  font-weight: 500;
}

.el-upload__tip {
  color: #909399;
  margin-top: 15px;
  font-size: 14px;
}

.detection-results {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.photo-preview {
  margin-bottom: 30px;
  max-width: 800px;
  transition: transform 0.3s ease;
}

.photo-preview:hover {
  transform: translateY(-5px);
}

.photo-preview img {
  max-width: 100%;
  border-radius: 12px;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
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

.upload-content {
  text-align: center;
  padding: 30px;
}

.upload-tip {
  color: #909399;
  font-size: 14px;
  margin-top: 10px;
}

.detection-report {
  border-radius: 12px;
}

.report-content {
  padding: 20px;
}

.report-summary {
  margin-bottom: 30px;
}

.stat-card {
  background: rgba(64, 158, 255, 0.1);
  padding: 20px;
  border-radius: 8px;
  text-align: center;
  transition: all 0.3s ease;
}

.stat-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.stat-value {
  font-size: 24px;
  font-weight: bold;
  color: #409EFF;
  margin-bottom: 8px;
}

.stat-label {
  font-size: 14px;
  color: #606266;
}

.preview-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 8px;
}

.el-carousel__item {
  border-radius: 8px;
  overflow: hidden;
}

.el-progress-bar__outer {
  background-color: rgba(64, 158, 255, 0.1);
}

.el-tabs__nav-wrap::after {
  background-color: rgba(64, 158, 255, 0.1);
}

@media (max-width: 768px) {
  .el-dialog {
    width: 90% !important;
  }
  
  .stat-card {
    margin-bottom: 15px;
  }
}
</style>
