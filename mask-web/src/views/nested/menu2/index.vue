<template>
  <div class="container-menu2">
    <div class="title" style="margin: left 100px;">修改密码</div>
    <el-form ref="form" :model="form" label-width="100px">
      <!-- <el-form-item label="原密码" style="width:300px;margin-right:80px;">
        <el-input v-model="form.old_password" />  
      </el-form-item> -->
      <el-form-item label="新密码" style="width:320px;" class="custom-label">
        <el-input v-model="form.new_password" />
      </el-form-item>
      <el-form-item label="确认密码" style="width:320px;" class="custom-label">
        <el-input v-model="form.new_confirm_password" />
      </el-form-item>
      <el-form-item>    
        <el-button type="primary" @click="onSubmit">确认修改</el-button>
        <el-button>取消重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
export default {
  data() {
    return {

      form: {
        new_password: '',
        new_confirm_password: ''
      }

    }
  },
  created() {
    const userInfo = JSON.parse(sessionStorage.getItem('userInfo'))
    // console.log('userInfo', userInfo)
    if (userInfo) {
      this.form = userInfo
    }
  },
  methods: {
    onSubmit() {
      let that = this;
      console.log('form-userInfo', this.form)
      const user_id = this.form.id;
      if(this.form.new_password!=this.form.new_confirm_password){

        this.$message({
       
          message: '两次密码不一致',
          type: 'error'

        })
        
        return
      }

      this.req({
        url: '/user/editPassword',
        method: 'get',
        params: {
          id: user_id,
          password:that.form.new_password
        }
      }).then(res => {
        console.log(res)

        this.$message({

          message: '密码修改成功',
          type: 'success'

        });

      })

    }
  }
}
</script>
<style >

.container-menu2{

  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 16% auto;
  max-width: 500px; 
  
  backdrop-filter: blur(10px); /* 毛玻璃效果 */
  border-radius: 15px; /* 圆角 */
  padding: 20px; /* 内边距 */ 
  box-shadow: 0 4px 10px rgba(3, 255, 209, 0.785); /* 阴影效果 */
}

.title{

  margin-bottom: 50px;
  font-size: 26px;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
  color: #9de5fd; /* 蓝紫色 */
  font-weight: bold; /* 添加加粗 */
}

.el-button {
  background-color: #22deec; /* 绿色按钮 */
  color: white; /* 白色文字 */
  border: none; /* 无边框 */
}

.el-button:hover {
  background-color: #388e46; /* 深绿色悬停效果 */
}

.el-form-item {
  margin-bottom: 20px; /* 表单项之间的间距 */
}

.custom-label .el-form-item__label {
  color: white ; /* 确保标签颜色为白色 */
}
</style>
