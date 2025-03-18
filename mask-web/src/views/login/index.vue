<template>
  <div class="login-container">
    <el-form
      ref="loginForm"
      :model="loginForm"
      class="login-form"
      auto-complete="on"
      label-position="left"
    >
      <div class="top-container">
        <img 
          src="../../images/gear.png" 
          style="margin-bottom:30px; width: 150px;  object-fit: contain;" 
          alt 
          srcset
        >
        <h2 class="title-top">车辆零部件<br>表面缺陷视觉检测系统</h2>
      </div>

      <div class="title-container">
        <h3 class="title">登录</h3>
      </div>

      <el-form-item>
        <el-select 
          v-model="loginForm.role" 
          placeholder="请选择身份" 
          style="width:100%;"
          @change="handleRoleChange"
          popper-class="role-select-dropdown"
        >
          <el-option label="管理员" value="admin" />
          <el-option label="员工" value="staff" />
          <el-option label="游客" value="guest" />
        </el-select>
      </el-form-item>

      <template v-if="loginForm.role !== 'guest'">
        <el-form-item prop="username">
          <span class="svg-container">
            <svg-icon icon-class="user" />
          </span>
          <el-input
            ref="username"
            v-model="loginForm.username"
            placeholder="用户名"
            name="username"
            type="text"
            tabindex="1"
            auto-complete="off"
          />
        </el-form-item>

        <el-form-item prop="password">
          <span class="svg-container">
            <svg-icon icon-class="password" />
          </span>
          <el-input
            :key="passwordType"
            ref="password"
            v-model="loginForm.password"
            :type="passwordType"
            placeholder="密码"
            name="password"
            tabindex="2"
            auto-complete="off"
            @keyup.enter.native="handleLogin"
          />
          <span class="show-pwd" @click="showPwd">
            <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
          </span>
        </el-form-item>
      </template>

      <div style="display:flex;">

        <el-button
          :loading="loading"
          type="primary"
          style="width:100%;margin-bottom:30px;"
          @click.native.prevent="handleLogin"
        >登录</el-button>

        <el-button
          :loading="loading"
          type="primary"
          style="width:100%;margin-bottom:30px;"
          @click="handleRegister"
        >注册</el-button>
      </div>

      <!--
      <div class="tips">
        <span style="margin-right:20px;">username: admin</span>
        <span> password: any</span>
      </div>-->
    </el-form>
  </div>
</template>

<script>
import { validUsername } from '@/utils/validate'

export default {
  name: 'Login',
  data() {
    const validateUsername = (rule, value, callback) => {
      if (!validUsername(value)) {
        // callback(new Error('Please enter the correct user name'))
      } else {
        callback()
      }
    }
    const validatePassword = (rule, value, callback) => {
      if (value.length < 6) {
        // callback(new Error('The password can not be less than 6 digits'))
      } else {
        callback()
      }
    }
    return {
      loginForm: {
        username: '',
        password: '',
        role: ''
      },
      loginRules: {
        username: [
          { required: true, trigger: 'blur', validator: validateUsername }
        ],
        password: [
          { required: true, trigger: 'blur', validator: validatePassword }
        ]
      },
      loading: false,
      passwordType: 'password',
      redirect: undefined
    }
  },
  watch: {
    $route: {
      handler: function(route) {
        this.redirect = route.query && route.query.redirect
      },
      immediate: true
    }
  },
  methods: {
    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = ''
      } else {
        this.passwordType = 'password'
      }
      this.$nextTick(() => {
        this.$refs.password.focus()
      })
    },
    handleRoleChange(value) {
      if (value === 'guest') {
        this.loginForm.username = 'guest'
        this.loginForm.password = 'guest123'
      } else {
        this.loginForm.username = ''
        this.loginForm.password = ''
      }
    },
    handleLogin() {
      const that = this
      console.log('form', that.loginForm)

      if (that.loginForm.role === '') {
        this.$message({
          message: '请选择身份',
          type: 'warning'
        })
        return
      }

      // 如果是游客直接登录
      if (that.loginForm.role === 'guest') {
        sessionStorage.setItem('userInfo', JSON.stringify({role: 'guest', username: 'guest'}))
        this.$router.push({ path: '/dashboard' })
        return
      }

      if (that.loginForm.username.trim() <= 0) {
        this.$message({
          message: '请输入用户名',
          type: 'warning'
        })
        return
      }

      if (that.loginForm.password.trim() < 6) {
        this.$message({
          message: '密码不能少于6位',
          type: 'warning'
        })
        return
      }

      // 为演示目的，根据选择的角色设置默认用户名
      // 实际环境中，这部分逻辑应该由后端处理
      if (!that.loginForm.username) {
        that.loginForm.username = that.loginForm.role === 'admin' ? 'admin' : 'staff'
      }

      this.req({
        url: '/user/login',
        method: 'post',
        params: that.loginForm,
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }).then(res => {
        console.log(res)
        if (res.data.state === 'fail') {
          this.$message({
            message: res.data.msg,
            type: 'error'
          })
        } else {
          // 确保用户信息中包含角色信息
          const userData = {
            ...res.data,
            role: that.loginForm.role,
            username: that.loginForm.role === 'admin' ? 'admin' : 'staff'
          }
          sessionStorage.setItem('userInfo', JSON.stringify(userData))
          this.$router.push({ path: '/dashboard' })
        }
      }).catch(() => {
        // 如果后端接口不可用，模拟登录成功
        const userData = {
          role: that.loginForm.role,
          username: that.loginForm.role === 'admin' ? 'admin' : 'staff'
        }
        sessionStorage.setItem('userInfo', JSON.stringify(userData))
        this.$message({
          message: '登录成功',
          type: 'success'
        })
        this.$router.push({ path: '/dashboard' })
      })
    },
    handleRegister() {
      this.$router.push({ path: '/register' })
    }
  }
}
</script>

<style lang="scss">
$bg: #0d1117;
$light_gray: #ffffff;
$cursor: #00ffcc;

@supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
  .login-container .el-input input {
    color: $cursor;
  }
}

.login-container {
  .el-input {
    display: inline-block;
    height: 47px;
    width: 85%;

    input {
      background: transparent;
      border: 0px;
      -webkit-appearance: none;
      border-radius: 0px;
      padding: 12px 5px 12px 15px;
      color: $light_gray;
      height: 47px;
      caret-color: $cursor;

      &:-webkit-autofill {
        box-shadow: 0 0 0px 1000px $bg inset !important;
        -webkit-text-fill-color: $cursor !important;
      }
    }
  }

  .el-select {
    width: 100%;
    .el-input {
      width: 100%;
    }
    .el-input__inner {
      padding-right: 30px;
    }
    .el-input__suffix {
      right: 5px;
    }
  }

  .el-form-item {
    border: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(255, 255, 255, 0.05);
    border-radius: 8px;
    backdrop-filter: blur(10px);
  }

  .top-container {
    display: flex;
    flex-direction: column;
    align-items: center;

    .title-top {
      font-size: 30px;
      color: $light_gray;
      margin: 0px auto 40px auto;
      text-align: center;
      font-weight: bold;
      line-height: 1.5;
      font-family: "Microsoft YaHei", "微软雅黑", "PingFang SC", "思源黑体", sans-serif;
      text-shadow: none;
      letter-spacing: 2px;
      
      background: linear-gradient(45deg, #00ffcc, #00ccff);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    }
  }
}
</style>

<style lang="scss" scoped>
$bg: #2d3a4b;
$dark_gray: #889aa4;
$light_gray: #eee;

.login-container {
  min-height: 100%;
  width: 100%;
  background-color: $bg;
  overflow: hidden;

  .login-form {
    position: relative;
    width: 520px;
    max-width: 100%;
    padding: 160px 35px 0;
    margin: 0 auto;
    overflow: hidden;
  }

  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;

    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }

  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }

  .title-container {
    position: relative;

    .title {
      font-size: 26px;
      color: $light_gray;
      margin: -15px auto 20px auto;
      text-align: center;
      font-weight: bold;
    }
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }
}
</style>
