<template>
  <div class="login">
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <img class="avatar" src="../assets/avatar.jpg" alt="">
        <el-form-item label="用户名" prop="username">
          <el-input placeholder="请输入用户名" v-model="form.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input @keyup.enter.native="login" placeholder="请输入密码" type="password" v-model="form.password"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button @click="login" class="login-btn" type="primary">登录</el-button>
          <el-button @click="reset" plain>重置</el-button>
        </el-form-item>
      </el-form>
  </div>
</template>

<script>
export default {
  data () {
    return {
      form: {
        username: '',
        password: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: ['blur', 'change'] },
          { min: 3, max: 12, message: '用户名长度在3到12位字符', trigger: ['blur', 'change'] }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: ['blur', 'change'] },
          { min: 3, max: 12, message: '密码长度在3到12位字符', trigger: ['blur', 'change'] }
        ]
      }
    }
  },
  methods: {
    reset () {
      this.$refs.form.resetFields()
    },
    async login () {
      try {
        await this.$refs.form.validate()
        const { meta, data } = await this.$axios.post('login', this.form)
        if (meta.status === 200) {
          localStorage.setItem('token', data.token)
          this.$message({
            type: 'success',
            message: '登陆成功!',
            duration: 2000
          })
          this.$router.push({ name: 'index' })
        } else {
          this.$message({
            type: 'error',
            message: meta.msg,
            duration: 2000
          })
        }
      } catch (e) {
        console.log(e)
      }
    }
  }
}
</script>

<style lang="scss">
  .login{
    width:100%;
    height:100%;
    background-color:#2D434C;
    overflow:hidden;
    .el-form {
      width:400px;
      padding:20px;
      padding-top:70px;
      background-color:white;
      margin:300px auto;
      position: relative;
      .avatar {
        position:absolute;
        left:50%;
        top:-50px;
        border-radius: 50%;
        border:5px solid white;
        transform:translateX(-50%)
      }
      .login-btn {
        margin-right:80px;
      }
    }
  }
</style>
