<template>
  <div class="container-photo">
    <div class="left-option">
      <el-form>
        <el-form-item label="模型选择">
          <el-select v-model="modules" placeholder="请选择合适的模型">
            <el-option label="yolov3-spp3(高准确度）" value="yolov3-spp3" />
            <el-option label="yolov3-tiny(高帧数)" value="yolov3-tiny" />
            <el-option label="mask-yolov5s" value="mask-yolov5s" />
            <el-option label="mask-yolov5m" value="mask-yolov5m" />
          </el-select>
        </el-form-item>
        <el-form-item style="margin-left:68px;">
          <el-button type="primary" style="width:200px;" @click="onSubmit">立即检测</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="right-present">
      <div v-if="photoUrls.length === 0">
        <el-upload
          class="upload-demo"
          drag
          action="http://localhost:81/api/file/photo"
          :on-preview="handlePreview"
          :on-remove="handleRemove"
          :on-success="handleSuccess"
          :before-upload="handleBefore"
          multiple
        >
          <i class="el-icon-upload" />
          <div class="el-upload__text">请将图片拖到此处，或<em>点击上传</em></div>
          <div slot="tip" class="el-upload__tip">上传jpg/png格式的图片</div>
        </el-upload>
      </div>
      <div v-if="photoUrls.length > 0" class="detection-results">
        <div v-for="(url, index) in photoUrls" :key="index">
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
        <el-button type="primary" style="width:200px;margin-top:30px" @click="onBack">重新上传</el-button>
      </div>
    </div>
  </div>
</template>
<!-- <script>
export default {
  data: function() {
    return {

      modules: '',
      photoUrl: '',
      imageName: '',
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
      const loading = this.$loading({
        lock: true,
        text: '检测中...',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      })

      this.req({
        url: '/file/checkphoto',
        methods: 'get',
        params: {
          model: that.modules,
          imageName: that.imageName
        }
      }).then(res => {
        console.log(res)
        this.photoUrl = res.data.imageUrl
        this.detectionTime = new Date().toLocaleString()
        this.detectionResults = {
          totalObjects: res.data.totalObjects || 0,
          confidence: res.data.confidence || '0%',
          details: res.data.details || []
        }
        loading.close()
        
        this.$message({
          message: '检测完成！',
          type: 'success'
        })
      }).catch(error => {
        loading.close()
        this.$message.error('检测失败，请重试')
      })
    },
    onBack() {
      this.photoUrl = ''
    },
    handleBefore() {

    },
    submitUpload() {
      this.$refs.upload.submit()
    },
    handleRemove(file, fileList) {
      console.log(file, fileList)
    },
    handlePreview(file) {
      // 显示预览功能
      console.log('预览文件:', file); // 添加调试信息
      const imageUrl = URL.createObjectURL(file.raw);
      console.log('生成的图片URL:', imageUrl); // 添加调试信息
      const imgWindow = window.open('', '_blank');
      imgWindow.document.write(`<img src="${imageUrl}" alt="Image Preview" style="width: 100%; height: auto;">`);
    },
    handleSuccess(res, file) {
      if (res.code === 200 || res.imageUrl) {
        this.photoUrls.push(res.imageUrl);
        this.imageNames.push(res.imageName);
        this.photoUrl = res.imageUrl;
        this.imageName = res.imageName;
        console.log('res', res);
        this.$message({
          message: '上传成功!',
          type: 'success'
        });
      } else {
        this.$message.error('上传失败，请重试！');
      }
    }
  }
} 
</script> -->
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
    
    handleSuccess(res, file) {
      if (res.code === 200 || res.imageUrl) {
        this.photoUrls.push(res.imageUrl);
        this.imageNames.push(res.imageName);
        this.photoUrl = res.imageUrl;
        this.imageName = res.imageName;
        
        // 添加预览逻辑
        const imageUrl = URL.createObjectURL(file); // 获取上传的文件的URL
        this.photoUrl = imageUrl; // 更新预览的图片URL

        this.$message({
          message: '上传成功!',
          type: 'success'
        });
      } else {
        this.$message.error('上传失败，请重试！')
      }
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
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.detection-results {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.result-card {
  width: 90%;
  margin-top: 20px;
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

/* 媒体查询以适应手机布局 */
@media (max-width: 768px) {
  .container-photo {
    flex-direction: column; /* 切换为垂直布局 */
    height: auto; /* 高度自适应 */
  }

  .left-option, .right-present {
    flex: none; /* 取消 flex 属性 */
    width: 100%; /* 宽度100% */
  }

  .result-card {
    width: 100%; /* 结果卡片宽度100% */
  }
}
</style>
