<template>
  <div class="login">
    <el-form ref="form" :model="form" :rules="rules" label-width="80px">
      <img src="../assets/avatar.jpg">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="form.username" placeholder="请输入用户名"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input v-model="form.password" placeholder="请输入密码" @keyup.enter.native="login('form')"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="login('form')">登陆</el-button>
        <el-button @click="reset('form')">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      form: {
        username: '',
        password: ''
      },
      rules: {
        username: [
          { required: true, message: '用户名不能为空', trigger: 'blur' },
          { min: 3, max: 9, message: '长度在 3 到 9 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '密码不能为空', trigger: 'blur' },
          { min: 6, max: 12, message: '长度在 6 到 12 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    login(formName) {
      this.$refs[formName].validate(async valid => {
        if (valid) {
          let res = await this.axios({
            url: 'login',
            method: 'post',
            data: this.form
          })
          if (res.meta.status === 200) {
            this.$message.success(res.meta.msg)
            localStorage.setItem('token', res.data.token)
            this.$router.push('/home')
          } else {
            this.$message({
              message: res.meta.msg,
              type: 'error'
            })
          }
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    reset(form) {
      this.$refs[form].resetFields()
    }
  }
}
</script>

<style lang="less" scoped>
.login {
  height: 100%;
  background-color: #2d434c;
  overflow: hidden;

  .el-form {
    width: 400px;
    padding: 75px 40px 15px;
    background-color: #fff;
    margin: 200px auto;
    border-radius: 20px;
    position: relative;

    img {
      position: absolute;
      top: -80px;
      left: 50%;
      transform: translateX(-50%);
      border-radius: 50%;
      border: 10px solid #fff;
    }

    .el-button + .el-button {
      margin-left: 70px;
    }
  }
}
</style>
