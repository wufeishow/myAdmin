<template>
  <div class="users">
    <el-table
      :data="list"
      style="width: 100%">
      <el-table-column prop="username" label="姓名">
      </el-table-column>
      <el-table-column prop="email" label="邮箱">
      </el-table-column>
      <el-table-column prop="mobile" label="电话">
      </el-table-column>
      <el-table-column label="用户状态">
        <template v-slot:default="obj">
          <el-switch
            @change="stateChange(obj.row.id,$event)"
            v-model="obj.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949">
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <el-button size="small" plain type="primary" icon="el-icon-edit"></el-button>
        <el-button size="small" plain type="danger" icon="el-icon-delete"></el-button>
        <el-button size="small" plain type="success" icon="el-icon-check">分配角色</el-button>
      </el-table-column>
    </el-table>
    <el-pagination
      background
      @current-change="pageChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </div>
</template>

<script>
import Axios from 'axios'
export default {
  created () {
    this.getList()
  },
  data () {
    return {
      list: [],
      query: '',
      pagenum: 1,
      pagesize: 2,
      total: 0
    }
  },
  methods: {
    getList () {
      Axios.get('http://localhost:8888/api/private/v1/users', {
        params: {
          query: this.query,
          pagenum: this.pagenum,
          pagesize: this.pagesize
        },
        headers: {
          Authorization: localStorage.getItem('token')
        }
      }).then(res => {
        console.log(res.data)
        const { meta, data } = res.data
        if (meta.status === 200) {
          this.list = data.users
          this.total = data.total
        }
      })
    },
    pageChange (page) {
      this.pagenum = page
      this.getList()
    },
    stateChange (id, state) {
      Axios.put(`http://localhost:8888/api/private/v1/users/${id}/state/${state}`, null, {
        headers: {
          Authorization: localStorage.getItem('token')
        }
      }).then(res => {
        if (res.data.meta.status === 200) {
          this.$message.success('修改成功')
        } else {
          this.$message.error('修改失败')
        }
      })
    }
  },
  watch: {

  }
}
</script>

<style lang="scss" scoped>
.users{
  .el-pagination {
    margin-top:5px;
  }
}
</style>
