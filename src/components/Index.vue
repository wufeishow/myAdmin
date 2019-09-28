<template>
  <el-container class="index">
    <el-header>
      <div class="logo">
        <img src="../assets/logo.png" alt="">
      </div>
      <div class="title">
        <h1>电商后台管理系统</h1>
      </div>
      <div class="logout">
        欢迎光临~<a @click="logout" href="javascript:;">退出</a>
      </div>
    </el-header>
    <el-container>
      <el-aside width="200px">
        <el-menu
          router
          unique-opened
          :default-active="getActive"
          class="el-menu-vertical-demo"
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b">
          <el-submenu :index="l1.path" v-for="l1 in menu" :key="l1.id">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>{{l1.authName}}</span>
            </template>
            <el-menu-item :index="l2.path" v-for="l2 in l1.children" :key="l2.id">
              <i class="el-icon-menu"></i>
              <span slot="title">{{l2.authName}}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main><router-view></router-view></el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  async created () {
    const { meta, data } = await this.$axios.get('menus')
    if (meta.status === 200) {
      this.menu = data
      console.log(data)
    } else {
      this.$message.error(meta.msg)
    }
  },
  data () {
    return {
      menu: []
    }
  },
  methods: {
    logout () {
      localStorage.removeItem('token')
      this.$router.push('/login')
    }
  },
  computed: {
    getActive () {
      return this.$route.path.slice(1)
    }
  }
}
</script>

<style lang="scss" scoped>
  .index{
    height:100%;
    .el-header {
      background-color:#D8D8D8;
      display:flex;

      .logo {
        width:180px;
        margin:10px;
        img {
          height:40px;
        }
      }
      .title {
        text-align:center;
        color:#545C64;
        height:60px;
        line-height:60px;
        flex:1;
      }
      .logout {
        text-align:right;
        height:60px;
        line-height:60px;
        font-weight:700;
        font-size:16px;
        a{
          color:orange;
        }
      }
    }

    .el-aside {
      background-color:#545C64;
      .el-menu {
        border-right:none;
      }
    }

    .el-main {
      background-color:#ECF0F1
    }
  }
</style>
