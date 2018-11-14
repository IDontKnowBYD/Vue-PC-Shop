<template>
  <el-container class="home">
    <el-header>
      <div class="img"></div>
      <div class="logout">欢迎光临<a @click="logout"> 退出</a></div>
      <h1 class="title">电商后台管理系统</h1>
    </el-header>
    <el-container>
      <el-aside width="200px">
        <el-menu
          class="el-menu-vertical-demo"
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b"
          unique-opened
          router
          >
          <el-submenu :index="menu.path" v-for="menu in menuList" :key="menu.id">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>{{menu.authName}}</span>
            </template>
            <el-menu-item :index="item.path" v-for="item in menu.children" :key="item.id">
              <i class="el-icon-menu"></i>{{item.authName}}
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      menuList: []
    }
  },
  methods: {
    async logout() {
      try {
        await this.$confirm('你确定要退出系统吗？', '温馨提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        this.$message({
          type: 'success',
          message: '退出成功!'
        })
        localStorage.removeItem('token')
        this.$router.push('/login')
      } catch (e) {
        this.$message({
          type: 'error',
          message: '取消了退出操作'
        })
      }
    }
  },
  async created() {
    let res = await this.axios.get('menus')
    let {
      meta: { status },
      data
    } = res
    if (status === 200) {
      this.menuList = data
    }
  }
}
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

    .el-submenu {
      width: 200px;
    }
  }

  .el-main {
    background-color: #d4dfe4;
  }
}
</style>
