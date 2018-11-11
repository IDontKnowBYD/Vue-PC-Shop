<template>
  <div class="users">
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/' }">用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <div>
      <el-input placeholder="请输入关键字" v-model="query" class="input-with-select">
        <el-button slot="append" icon="el-icon-search" @click="search"></el-button>
      </el-input>
      <el-button type="success" plain @click="addDialogVisible = true">用户添加</el-button>
    </div>
    <el-table
      :data="tableData"
      style="width: 100%">
      <el-table-column
        prop="username"
        label="姓名"
        width="180">
      </el-table-column>
      <el-table-column
        prop="email"
        label="邮箱"
        width="180">
      </el-table-column>
      <el-table-column
        prop="mobile"
        label="电话"
        width="180">
      </el-table-column>
      <el-table-column
        label="状态"
        width="180">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
            @change="stateChange(scope.row)">
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column
        label="操作">
        <template slot-scope="scope">
          <el-button plain type="primary" icon="el-icon-edit" size="small" @click="openChangeDialog(scope.row)"></el-button>
          <el-button plain type="danger" icon="el-icon-delete" size="small" @click="delUser(scope.row.id)"></el-button>
          <el-button plain type="success" icon="el-icon-check" size="small">分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="block">
      <el-pagination
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        :page-size="pageSize"
        :page-sizes="[2, 4, 6, 8]"
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        >
      </el-pagination>
    </div>
    <!-- 用户添加框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="40%">
      <!-- 添加表单 -->
      <el-form ref="addForm" :model="addForm" label-width="80px" :rules="rules">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" placeholder="请输入密码"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email" placeholder="请输入邮箱"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="addForm.mobile" placeholder="请输入电话"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser('addForm')">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="changeDialogVisible"
      width="40%">
      <!-- 添加表单 -->
      <el-form ref="changeForm" :model="changeForm" label-width="80px" :rules="rules">
        <el-form-item label="用户名" prop="username">
          <el-tag type="info">{{changeForm.username}}</el-tag>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="changeForm.email" placeholder="请输入邮箱"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="changeForm.mobile" placeholder="请输入电话"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="changeDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="changeUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tableData: [],
      total: 0,
      pageSize: 2,
      currentPage: 1,
      query: '',
      addDialogVisible: false,
      changeDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      changeForm: {
        id: '',
        username: '',
        email: '',
        mobile: ''
      },
      rules: {
        username: [
          { required: true, message: '用户名不能为空', trigger: 'blur' },
          { min: 3, max: 9, message: '长度在 3 到 9 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '密码不能为空', trigger: 'blur' },
          { min: 6, max: 12, message: '长度在 6 到 12 个字符', trigger: 'blur' }
        ],
        email: {
          type: 'email',
          message: '请输入正确格式的邮箱',
          trigger: 'blur'
        },
        mobile: {
          pattern: /^1\d{10}$/,
          message: '请输入正确的手机号',
          trigger: 'blur'
        }
      }
    }
  },
  created() {
    this.setCurrentPage()
  },
  methods: {
    setCurrentPage() {
      this.axios({
        url: 'users',
        method: 'get',
        params: {
          query: this.query,
          pagenum: this.currentPage,
          pagesize: this.pageSize
        }
      }).then(res => {
        if (res.meta.status === 200) {
          // console.log(res.data.users)
          this.tableData = res.data.users
          this.total = res.data.total
        }
      })
    },
    handleSizeChange(val) {
      this.pageSize = val
      this.setCurrentPage()
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.setCurrentPage()
    },
    // 搜索框
    search() {
      this.currentPage = 1
      this.setCurrentPage()
    },
    // 状态改变
    stateChange(val) {
      // console.log(val)
      let id = val.id
      let type = val.mg_state
      this.axios({
        url: `users/${id}/state/${type}`,
        method: 'put'
      }).then(res => {
        if (res.meta.status === 200) {
          this.$message({
            message: res.meta.msg,
            type: 'success'
          })
        }
      })
    },
    // 添加用户
    addUser(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.axios({
            url: 'users',
            method: 'post',
            data: this.addForm
          }).then(res => {
            this.addDialogVisible = false
            this.$refs[formName].resetFields()
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    // 删除用户
    delUser(id) {
      this.$confirm('你确定要删除这个用户吗？', '警告', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.axios({
            url: `users/${id}`,
            method: 'delete'
          }).then(res => {
            if (res.meta.status === 200) {
              if ((this.tableData.length = 1 && this.currentPage > 1)) {
                this.currentPage--
              }
              this.setCurrentPage()

              this.$message({
                type: 'success',
                message: '删除成功!'
              })
            }
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 修改用户
    // 打开修改框
    openChangeDialog(data) {
      this.changeForm.id = data.id
      this.changeForm.username = data.username
      this.changeForm.email = data.email
      this.changeForm.mobile = data.mobile
      this.changeDialogVisible = true
    },
    // 进行修改
    changeUser() {
      this.$refs.changeForm.validate(valid => {
        if (!valid) return false

        this.axios({
          url: `users/${this.changeForm.id}`,
          method: 'put',
          data: this.changeForm
        }).then(res => {
          if (res.meta.status === 200) {
            this.setCurrentPage()

            this.$message({
              type: 'success',
              message: '修改成功!'
            })

            this.changeDialogVisible = false
          }
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
.el-breadcrumb {
  height: 30px;
  line-height: 30px;
}

.el-input {
  width: 400px;
  margin-bottom: 10px;
}
</style>
