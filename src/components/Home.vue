<template>
  <el-container class="home">
    <el-header>
      <div class="img"></div>
      <div class="logout">欢迎光临<a> 退出</a></div>
      <h1 class="title">电商后台管理系统</h1>
    </el-header>
    <el-container>
      <el-aside width="200px">Aside</el-aside>
      <el-main>Main</el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {}
</script>

<style lang="less">
.home {
  height: 100%;

  .el-header {
    background-color: #b3c1cd;

    .img {
      width: 180px;
      height: 60px;
      float: left;
      background: url('../assets/logo.png') no-repeat;
      background-size: contain;
      background-position: center center;
    }
    .logout {
      width: 180px;
      height: 60px;
      float: right;
      line-height: 60px;
      text-align: right;
      font-weight: 700;

      a {
        color: darkorange;
      }
    }
    .title {
      height: 60px;
      overflow: hidden;
      text-align: center;
      line-height: 60px;
      color: #fff;
      font-size: 30px;
    }
  }

  .el-aside {
    background-color: #545c64;
  }

  .el-main {
    background-color: #d4dfe4;
  }
}
</style>
