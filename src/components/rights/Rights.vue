<template>
  <div class="rights">
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/' }">权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-button type="success" plain @click="addDialogVisible = true">添加角色</el-button>
    <!-- 表格 -->
     <el-table
      :data="tableData"
      style="width: 100%">
       <el-table-column
          type="index">
        </el-table-column>
      <el-table-column
        prop="authName"
        label="权限名称">
      </el-table-column>
      <el-table-column
        prop="path"
        label="路径">
      </el-table-column>
      <el-table-column
        label="层级">
        <template slot-scope="scope">
          <span v-if="scope.row.level === '0'">一级</span>
          <span v-else-if="scope.row.level === '1'">二级</span>
          <span v-else>三级</span>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tableData: []
    }
  },
  async created() {
    let res = await this.axios.get('rights/list')

    if (res.meta.status === 200) {
      this.tableData = res.data
    }
  }
}
</script>

<style>
</style>
