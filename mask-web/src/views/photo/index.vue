<template>
  <div class="container-photo">
    <div class="main-layout">
      <!-- 左侧功能面板 -->
      <div class="function-panel">
        <h3 class="panel-title">功能选择</h3>
        <div class="function-buttons">
          <el-select v-model="selectedModel" placeholder="选择模型" class="model-select">
            <el-option label="net_normal(高准确度)" value="net_normal" />
            <el-option label="net_light(高速度)" value="net_light" />
          </el-select>

          <el-button type="primary" @click="openFile">
            <i class="el-icon-folder-opened"></i> 打开图片
          </el-button>
          
          <el-button type="primary" @click="batchTest">
            <i class="el-icon-files"></i> 批量测试
          </el-button>
          
          <el-button type="success" @click="startDetection" :disabled="!previewImage || isProcessing">
            <i class="el-icon-video-play"></i> 开始检测
          </el-button>
          
          <el-button type="warning" @click="stopDetection" :disabled="!isProcessing">
            <i class="el-icon-video-pause"></i> 停止检测
          </el-button>
          
          <el-button type="info" @click="savePath">
            <i class="el-icon-folder"></i> 保存路径
          </el-button>
        </div>
      </div>

      <!-- 中间内容区域 -->
      <div class="content-area">
        <!-- 待测图片区域 -->
        <div class="image-section">
          <h3 class="section-title">待测图片</h3>
          <div class="image-container" @click="openFile">
            <template v-if="previewImage">
              <div class="image-wrapper">
                <img :src="previewImage" alt="预览图片">
              </div>
            </template>
            <div v-else class="upload-placeholder">
              <i class="el-icon-plus"></i>
              <p>点击添加图片</p>
            </div>
          </div>
        </div>

        <!-- 检测结果区域 -->
        <div class="result-section">
          <h3 class="section-title">检测结果</h3>
          <div class="result-container">
            <template v-if="detectionResult">
              <div class="image-wrapper">
                <img :src="detectionResult" alt="检测结果">
              </div>
            </template>
            <div v-else-if="isProcessing" class="processing-container">
              <el-progress
                type="circle"
                :percentage="processPercentage"
                :status="processStatus"
                class="progress-circle"
              />
              <p class="processing-text">{{ processText }}</p>
            </div>
            <div v-else class="result-placeholder">
              <i class="el-icon-warning-outline"></i>
              <p>暂无检测结果</p>
            </div>
          </div>
        </div>
      </div>

      <!-- 右侧历史记录 -->
      <div class="history-panel">
        <h3 class="panel-title">历史记录</h3>
        <div class="history-list">
          <el-timeline>
            <el-timeline-item
              v-for="(history, index) in historyList"
              :key="index"
              :timestamp="history.time"
              placement="top"
            >
              <div class="history-item">
                <img :src="history.thumbnail" alt="缩略图" class="history-thumbnail">
                <div class="history-info">
                  <p>检测结果：{{ history.result }}</p>
                  <p>置信度：{{ history.confidence }}</p>
                </div>
              </div>
            </el-timeline-item>
          </el-timeline>
        </div>
      </div>
    </div>

    <!-- 隐藏的文件上传输入框 -->
    <input 
      type="file" 
      ref="fileInput" 
      style="display: none" 
      accept="image/*" 
      @change="handleFileChange"
    />
    
    <!-- 隐藏的文件夹上传输入框 -->
    <input 
      type="file" 
      ref="folderInput" 
      style="display: none" 
      webkitdirectory 
      multiple 
      @change="handleFolderChange"
    />

    <!-- 批量测试对话框 -->
    <el-dialog
      title="批量测试"
      :visible.sync="batchDialogVisible"
      width="50%"
    >
      <div class="batch-dialog-content">
        <div class="batch-upload-area" @click="triggerFolderUpload">
          <i class="el-icon-upload"></i>
          <p>点击选择文件夹</p>
          <p class="upload-tip">已选择 {{ batchFiles.length }} 个文件</p>
        </div>
        
        <div class="batch-file-list" v-if="batchFiles.length > 0">
          <p>已选择文件：</p>
          <el-table :data="batchFilesPreview" height="200">
            <el-table-column prop="name" label="文件名"></el-table-column>
            <el-table-column prop="size" label="大小" width="120"></el-table-column>
          </el-table>
        </div>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="batchDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="startBatchTest" :disabled="batchFiles.length === 0">开始测试</el-button>
      </span>
    </el-dialog>

    <!-- 停止检测对话框 -->
    <el-dialog
      title="停止检测"
      :visible.sync="stopDialogVisible"
      width="30%"
    >
      <p>确定要停止当前检测任务吗？</p>
      <span slot="footer" class="dialog-footer">
        <el-button @click="stopDialogVisible = false">取消</el-button>
        <el-button type="danger" @click="confirmStopDetection">确定停止</el-button>
      </span>
    </el-dialog>

    <!-- 保存路径对话框 -->
    <el-dialog
      title="设置保存路径"
      :visible.sync="savePathDialogVisible"
      width="40%"
    >
      <el-form :model="savePathForm" label-width="100px">
        <el-form-item label="保存路径">
          <el-input v-model="savePathForm.path" placeholder="请输入保存路径">
            <el-button slot="append" icon="el-icon-folder" @click="browsePath">浏览</el-button>
          </el-input>
        </el-form-item>
        <el-form-item label="文件格式">
          <el-select v-model="savePathForm.format" placeholder="请选择文件格式">
            <el-option label="JPG" value="jpg"></el-option>
            <el-option label="PNG" value="png"></el-option>
            <el-option label="TIFF" value="tiff"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="自动保存">
          <el-switch v-model="savePathForm.autoSave"></el-switch>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="savePathDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="confirmSavePath">确定</el-button>
      </span>
    </el-dialog>

    <!-- 检测完成结果对话框 -->
    <el-dialog
      title="检测完成"
      :visible.sync="resultDialogVisible"
      width="60%"
      class="result-dialog"
    >
      <div class="result-dialog-content">
        <div class="result-summary">
          <div class="result-item">
            <div class="result-label">检测结果</div>
            <div class="result-value">{{ detectionSummary.result }}</div>
          </div>
          <div class="result-item">
            <div class="result-label">置信度</div>
            <div class="result-value">{{ detectionSummary.confidence }}</div>
          </div>
          <div class="result-item">
            <div class="result-label">处理时间</div>
            <div class="result-value">{{ detectionSummary.time }}ms</div>
          </div>
        </div>
        
        <div class="result-details">
          <h4>缺陷详情</h4>
          <el-table :data="detectionSummary.defects" height="200">
            <el-table-column prop="type" label="缺陷类型"></el-table-column>
            <el-table-column prop="count" label="数量" width="80"></el-table-column>
            <el-table-column prop="confidence" label="置信度" width="100"></el-table-column>
            <el-table-column prop="severity" label="严重程度" width="100">
              <template slot-scope="scope">
                <el-tag :type="getSeverityType(scope.row.severity)">{{ scope.row.severity }}</el-tag>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="resultDialogVisible = false">关闭</el-button>
        <el-button type="primary" @click="saveResult">保存结果</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedModel: 'net_normal',
      previewImage: '',
      detectionResult: '',
      isProcessing: false,
      processPercentage: 0,
      processStatus: '',
      processText: '正在处理...',
      processTimer: null,
      
      // 批量测试相关
      batchDialogVisible: false,
      batchFiles: [],
      batchFilesPreview: [],
      
      // 停止检测相关
      stopDialogVisible: false,
      
      // 保存路径相关
      savePathDialogVisible: false,
      savePathForm: {
        path: 'D:/Desktop/检测结果',
        format: 'jpg',
        autoSave: false
      },
      
      // 检测结果相关
      resultDialogVisible: false,
      detectionSummary: {
        result: '发现缺陷',
        confidence: '95.8%',
        time: 1250,
        defects: [
          { type: '表面裂纹', count: 3, confidence: '96.5%', severity: '严重' },
          { type: '材料变形', count: 1, confidence: '92.3%', severity: '中等' },
          { type: '表面划痕', count: 2, confidence: '89.7%', severity: '轻微' }
        ]
      },
      
      // 历史记录
      historyList: [
        {
          time: '2024-03-14 15:30',
          thumbnail: 'https://via.placeholder.com/60x60',
          result: '表面裂纹',
          confidence: '95.6%'
        },
        {
          time: '2024-03-14 15:25',
          thumbnail: 'https://via.placeholder.com/60x60',
          result: '材料变形',
          confidence: '92.3%'
        }
      ],
      postProcessPercentage: 0,
      postProcessTimer: null,
    }
  },
  methods: {
    // 打开单个图片
    openFile() {
      this.$refs.fileInput.click();
    },
    
    // 处理单个图片选择
    handleFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        if (file.type.startsWith('image/')) {
          this.previewImage = URL.createObjectURL(file);
          this.detectionResult = ''; // 清空之前的检测结果
          this.$message.success('图片上传成功');
        } else {
          this.$message.error('请上传图片文件');
        }
      }
      // 重置input，以便可以重复选择同一文件
      this.$refs.fileInput.value = '';
    },
    
    // 批量测试
    batchTest() {
      this.batchDialogVisible = true;
      this.batchFiles = [];
      this.batchFilesPreview = [];
    },
    
    // 触发文件夹选择
    triggerFolderUpload() {
      this.$refs.folderInput.click();
    },
    
    // 处理文件夹选择
    handleFolderChange(event) {
      const files = Array.from(event.target.files);
      this.batchFiles = files.filter(file => file.type.startsWith('image/'));
      
      // 生成预览数据
      this.batchFilesPreview = this.batchFiles.slice(0, 10).map(file => {
        return {
          name: file.name,
          size: this.formatFileSize(file.size)
        };
      });
      
      if (this.batchFiles.length > 0) {
        this.$message.success(`已选择 ${this.batchFiles.length} 个图片文件`);
      } else {
        this.$message.warning('所选文件夹中没有图片文件');
      }
      
      // 重置input，以便可以重复选择同一文件夹
      this.$refs.folderInput.value = '';
    },
    
    // 格式化文件大小
    formatFileSize(size) {
      if (size < 1024) {
        return size + ' B';
      } else if (size < 1024 * 1024) {
        return (size / 1024).toFixed(2) + ' KB';
      } else {
        return (size / (1024 * 1024)).toFixed(2) + ' MB';
      }
    },
    
    // 开始批量测试
    startBatchTest() {
      this.batchDialogVisible = false;
      
      const loading = this.$loading({
        lock: true,
        text: `正在处理 ${this.batchFiles.length} 个文件...`,
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      });
      
      // 模拟处理过程
      setTimeout(() => {
        loading.close();
        this.$message.success('批量测试完成');
        
        // 更新历史记录
        const now = new Date();
        const timeStr = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')} ${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}`;
        
        // 添加批量测试结果到历史记录
        this.historyList.unshift({
          time: timeStr,
          thumbnail: 'https://via.placeholder.com/60x60',
          result: '批量测试',
          confidence: '93.2%'
        });
      }, 3000);
    },
    
    // 开始检测
    startDetection() {
      if (!this.previewImage) {
        this.$message.warning('请先上传图片');
        return;
      }
      
      this.isProcessing = true;
      this.processPercentage = 0;
      this.processStatus = '';
      this.processText = '正在处理...';
      this.detectionResult = '';
      
      // 模拟进度更新
      this.processTimer = setInterval(() => {
        this.processPercentage += 2;
        
        if (this.processPercentage >= 100) {
          clearInterval(this.processTimer);
          this.processStatus = 'success';
          this.processText = '处理完成';
          
          // 模拟检测结果
          setTimeout(() => {
            this.isProcessing = false;
            this.detectionResult = 'https://via.placeholder.com/800x400';
            
            // 更新历史记录
            const now = new Date();
            const timeStr = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')} ${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}`;
            
            this.historyList.unshift({
              time: timeStr,
              thumbnail: this.previewImage,
              result: '检测完成',
              confidence: '95.8%'
            });
          }, 500);
        }
      }, 100);
    },
    
    // 停止检测
    stopDetection() {
      if (this.isProcessing) {
        this.stopDialogVisible = true;
      }
    },
    
    // 确认停止检测
    confirmStopDetection() {
      this.stopDialogVisible = false;
      
      if (this.processTimer) {
        clearInterval(this.processTimer);
      }
      
      this.isProcessing = false;
      this.processPercentage = 0;
      this.$message.info('检测已停止');
    },
    
    // 保存路径
    savePath() {
      this.savePathDialogVisible = true;
    },
    
    // 浏览路径
    browsePath() {
      // 创建一个隐藏的 input 元素用于选择文件夹
      const input = document.createElement('input');
      input.type = 'file';
      input.webkitdirectory = true;
      input.style.display = 'none';
      
      input.onchange = (event) => {
        if (event.target.files.length > 0) {
          // 获取选择的文件夹路径（仅显示第一个文件所在的目录）
          const path = event.target.files[0].webkitRelativePath.split('/')[0];
          this.savePathForm.path = `D:/Projects/${path}`;
          this.$message.success('路径选择成功');
        }
      };

      // 触发文件夹选择
      document.body.appendChild(input);
      input.click();
      document.body.removeChild(input);
    },
    
    // 确认保存路径
    confirmSavePath() {
      this.savePathDialogVisible = false;
      this.$message.success('保存路径设置成功');
    },
    
    // 保存检测结果
    saveResult() {
      this.resultDialogVisible = false;
      this.$message.success('检测结果已保存');
    },
    
    // 获取严重程度对应的标签类型
    getSeverityType(severity) {
      switch (severity) {
        case '严重':
          return 'danger';
        case '中等':
          return 'warning';
        case '轻微':
          return 'info';
        default:
          return '';
      }
    },
    format(percentage) {
      return percentage === 100 ? '完成' : `${percentage}%`;
    }
  }
}
</script> 

<style scoped>
.main-layout {
  display: grid;
  grid-template-columns: 220px 1fr 250px;
  gap: 20px;
  padding: 20px;
  height: calc(100vh - 40px);
  background: #f5f7fa;
}

.function-panel {
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 12px 0 rgba(0,0,0,0.05);
  display: flex;
  flex-direction: column;
  height: 100%;
}

.panel-title {
  color: #303133;
  font-size: 16px;
  margin-bottom: 20px;
  padding-bottom: 10px;
  border-bottom: 1px solid #ebeef5;
}

.function-buttons {
  display: flex;
  flex-direction: column;
  gap: 15px;
  padding: 0;
}

.function-buttons .el-button {
  width: 100%;
  height: 40px;
  margin: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
}

.function-buttons .el-button i {
  margin-right: 8px;
  font-size: 16px;
}

.model-select {
  width: 100%;
}

.function-buttons .el-select {
  width: 100%;
}

/* 确保下拉选择框的高度与按钮一致 */
.function-buttons .el-input__inner {
  height: 40px;
  line-height: 40px;
}

/* 移除按钮的默认外边距 */
.el-button + .el-button {
  margin-left: 0;
}

/* 确保图标对齐 */
.el-button [class*="el-icon-"] + span {
  margin-left: 0;
}

.content-area {
  display: grid;
  grid-template-rows: 1fr 1fr;
  gap: 20px;
  min-height: 0; /* 防止内容溢出 */
  overflow: hidden; /* 确保内容不会溢出 */
}

.image-section, .result-section {
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 12px 0 rgba(0,0,0,0.05);
  display: flex;
  flex-direction: column;
  height: 100%;
  min-height: 0; /* 防止内容溢出 */
}

.section-title {
  color: #303133;
  font-size: 16px;
  margin-bottom: 15px;
}

.image-container, .result-container {
  flex: 1;
  border: 2px dashed #dcdfe6;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  transition: all 0.3s ease;
  position: relative;
  background-color: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  min-height: 0; /* 防止内容溢出 */
}

.image-wrapper {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  padding: 10px;
  min-height: 0; /* 防止内容溢出 */
}

.image-container:hover {
  border-color: #409EFF;
  cursor: pointer;
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(64, 158, 255, 0.2);
}

.image-container img, .result-container img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  width: auto;
  height: auto;
}

.upload-placeholder, .result-placeholder {
  text-align: center;
  color: #909399;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
}

.upload-placeholder i, .result-placeholder i {
  font-size: 60px;
  margin-bottom: 20px;
  color: #409EFF;
  transition: transform 0.3s ease;
}

.image-container:hover .upload-placeholder i {
  transform: scale(1.1);
}

.processing-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
}

.processing-text {
  margin-top: 15px;
  color: #409EFF;
  font-size: 14px;
}

.progress-circle {
  margin: 20px 0;
}

.history-list {
  height: calc(100% - 40px);
  overflow-y: auto;
}

.history-item {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

.history-thumbnail {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 4px;
}

.history-info {
  font-size: 14px;
}

.history-info p {
  margin: 5px 0;
  color: #606266;
}

/* 批量测试对话框样式 */
.batch-dialog-content {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.batch-upload-area {
  border: 2px dashed #dcdfe6;
  border-radius: 6px;
  padding: 30px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
}

.batch-upload-area:hover {
  border-color: #409EFF;
}

.batch-upload-area i {
  font-size: 48px;
  color: #409EFF;
  margin-bottom: 15px;
}

.upload-tip {
  margin-top: 10px;
  color: #909399;
  font-size: 14px;
}

.batch-file-list {
  margin-top: 20px;
}

/* 结果对话框样式 */
.result-dialog-content {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.result-summary {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.result-item {
  text-align: center;
  padding: 15px;
  background-color: #f5f7fa;
  border-radius: 6px;
  flex: 1;
  margin: 0 10px;
}

.result-label {
  font-size: 14px;
  color: #606266;
  margin-bottom: 10px;
}

.result-value {
  font-size: 20px;
  font-weight: bold;
  color: #303133;
}

.result-details h4 {
  margin-bottom: 15px;
  color: #303133;
}

/* 主题色调整 */
.el-button--primary {
  background-color: #409EFF;
  border-color: #409EFF;
}

.el-button--success {
  background-color: #67C23A;
  border-color: #67C23A;
}

.el-button--warning {
  background-color: #E6A23C;
  border-color: #E6A23C;
}

.el-button--info {
  background-color: #909399;
  border-color: #909399;
}

.el-button--danger {
  background-color: #F56C6C;
  border-color: #F56C6C;
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .main-layout {
    grid-template-columns: 200px 1fr 220px;
  }
}

@media (max-width: 992px) {
  .main-layout {
    grid-template-columns: 1fr;
    grid-template-rows: auto 1fr auto;
  }
  
  .result-summary {
    flex-direction: column;
    gap: 10px;
  }
  
  .result-item {
    margin: 0;
  }
}
</style>
