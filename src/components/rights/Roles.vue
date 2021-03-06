<template>
  <div class="roles">
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/' }">角色管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-button type="success" plain @click="addDialogVisible = true">添加角色</el-button>
    <!-- 表格 -->
     <el-table
      :data="tableData"
      style="width: 100%">
      <el-table-column type="expand">
        <template slot-scope="scope">
          <el-row  class="level1" v-for="level1 in scope.row.children" :key="level1.id">
            <el-col :span="4"><el-tag closable @close="deleteRight(scope.row, level1.id)">{{level1.authName}}</el-tag></el-col>
            <el-col :span="20">
              <el-row class="level2" v-for="level2 in level1.children" :key="level2.id">
                <el-col :span="4"><el-tag closable type="success" @close="deleteRight(scope.row, level2.id)">{{level2.authName}}</el-tag></el-col>
                <el-col :span="20">
                  <el-tag class="level3" closable type="danger" @close="deleteRight(scope.row, level3.id)" v-for="level3 in level2.children" :key="level3.id">
                    {{level3.authName}}
                  </el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <el-table-column
        type="index">
      </el-table-column>
      <el-table-column
        prop="roleName"
        label="角色名称">
      </el-table-column>
      <el-table-column
        prop="roleDesc"
        label="角色描述">
      </el-table-column>
      <el-table-column
        label="操作">
        <template slot-scope="scope">
          <el-button plain type="primary" icon="el-icon-edit" size="small" @click="showEditDialog(scope.row)"></el-button>
          <el-button plain type="danger" icon="el-icon-delete" size="small" @click="deleteRole(scope.row)"></el-button>
          <el-button plain type="success" icon="el-icon-check" size="small" @click="showAssignDialog(scope.row)">分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 权限弹出框 -->
    <el-dialog
      title="提示"
      :visible.sync="assignDialogVisible"
      width="30%">
      <el-tree
        ref="tree"
        :data="rightList"
        show-checkbox
        node-key="id"
        default-expand-all
        :props="defaultProps">
      </el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="assignDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="assignRight">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加弹出框 -->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="40%">
      <el-form :model="addForm" label-width="80px" :rules="rules" status-icon ref="addForm">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 修改弹出框 -->
    <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="40%">
      <el-form :model="editForm" label-width="80px" :rules="rules" status-icon ref="editForm">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tableData: [],
      assignDialogVisible: false,
      rightList: [],
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      roleId: '',
      addDialogVisible: false,
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      rules: {
        roleName: [
          { required: true, message: '角色名称不能为空', trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      editForm: {
        roleName: '',
        roleDesc: '',
        id: ''
      }
    }
  },
  methods: {
    async getRoleList() {
      let res = await this.axios.get('roles')

      if (res.meta.status === 200) {
        this.tableData = res.data
      }
    },
    // 删除权限
    async deleteRight(role, rightId) {
      let res = await this.axios.delete(`roles/${role.id}/rights/${rightId}`)

      let {
        meta: { status },
        data
      } = res
      if (status === 200) {
        role.children = data
        this.$message.success('恭喜你，删除成功')
      }
    },
    // 展示权限列表
    async showAssignDialog(role) {
      let res = await this.axios.get(`rights/tree`)
      console.log(res)
      this.rightList = res.data
      this.roleId = role.id

      this.assignDialogVisible = true

      this.$nextTick(() => {
        let temp = []
        role.children.forEach(l1 => {
          l1.children.forEach(l2 => {
            l2.children.forEach(l3 => {
              temp.push(l3.id)
            })
          })
        })

        this.$refs.tree.setCheckedKeys(temp)
      })
    },
    // 分配权限
    async assignRight() {
      let checked = this.$refs.tree.getCheckedKeys()
      let halfChecked = this.$refs.tree.getHalfCheckedKeys()
      let all = [...checked, ...halfChecked]

      let res = await this.axios.post(`roles/${this.roleId}/rights`, {
        rids: all.join()
      })

      if (res.meta.status === 200) {
        this.assignDialogVisible = false
        this.getRoleList()
      }
    },
    // 删除角色
    async deleteRole(role) {
      try {
        await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        let res = await this.axios.delete(`roles/${role.id}`)

        if (res.meta.status === 200) {
          this.getRoleList()

          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        }
      } catch (e) {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      }
    },
    // 添加角色
    async addRole() {
      this.$refs.addForm.validate(async valid => {
        if (!valid) return false

        let res = await this.axios.post('roles', this.addForm)

        if (res.meta.status === 201) {
          this.addDialogVisible = false

          this.$message({
            type: 'success',
            message: '添加成功!'
          })

          this.getRoleList()

          this.$refs.addForm.resetFields()
        }
      })
    },
    // 展示修改框
    showEditDialog(role) {
      this.editDialogVisible = true

      this.editForm.roleName = role.roleName
      this.editForm.roleDesc = role.roleDesc
      this.editForm.id = role.id
    },
    // 提交修改
    editRole() {
      this.$refs.editForm.validate(async valid => {
        if (!valid) return false

        let res = await this.axios.put(
          `roles/${this.editForm.id}`,
          this.editForm
        )

        if (res.meta.status === 200) {
          this.editDialogVisible = false

          this.$message({
            type: 'success',
            message: '修改成功!'
          })

          this.getRoleList()
        }
      })
    }
  },
  created() {
    this.getRoleList()
  }
}
</script>

<style lang="less" scoped>
.level2 {
  margin-bottom: 10px;
}

.level3 {
  margin-right: 10px;
  margin-bottom: 5px;
}
</style>
