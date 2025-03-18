<template>
  <div class="container">
    <div class="title">用户信息</div>
    <el-form ref="form" :model="form" label-width="100px">
      <el-form-item label="用户名" style="width:300px;"class="custom-label">
        <el-input v-model="form.username" />
      </el-form-item>
      <el-form-item label="性别" style="width:360px;"class="custom-label">
        <el-select v-model="form.gender" placeholder="请选择性别">
          <el-option label="男" value="男" />
          <el-option label="女" value="女" />
        </el-select>
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
        username: 'CFPL',
        gender: '男'
      }
    }
  },
  created() {
    const userInfo = JSON.parse(sessionStorage.getItem('userInfo'))
    if (userInfo) {
      this.form = userInfo
    }
  },
  methods: {
    onSubmit() {
      // const that = this
      console.log('form-userInfo', this.form)
      const username = this.form.username
      const user_id = this.form.id
      const gender = this.form.gender
      this.req({
        url: '/user/editUserWeb',
        method: 'get',
        params: {
          id: user_id,
          username: username,
          gender: gender
        }
      }).then(res => {
        console.log(res)

        this.$message({

          message: '信息修改成功',
          type: 'success'

        });

      })
    }
  }
}
</script>
<style >

.container{

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
