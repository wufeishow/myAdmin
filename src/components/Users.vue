<template>
  <div class="users">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <div class="search-bar">
      <el-input placeholder="请输入内容" v-model="query" class="input-with-select search-input">
        <el-button @click="search" slot="append" icon="el-icon-search"></el-button>
      </el-input>
      <el-button @click="showCreate" class="create-btn" type="success" plain>添加用户</el-button>
    </div>

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
        <template v-slot:default="{row}">
          <el-button @click="showEdit(row)" size="small" plain type="primary" icon="el-icon-edit"></el-button>
          <el-button @click="del(row.id)" size="small" plain type="danger" icon="el-icon-delete"></el-button>
          <el-button @click="showAssign(row)" size="small" plain type="success" icon="el-icon-check">分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      background
      @size-change="sizeChange"
      @current-change="pageChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>

    <el-dialog
      title="提示"
      :visible.sync="createVisible"
      width="40%">
      <el-form :rules="rules" ref="createForm" :model="form" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="form.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input type="password" v-model="form.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="form.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="form.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="createVisible = false">取 消</el-button>
        <el-button @click="create" type="primary">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog
      title="提示"
      :visible.sync="editVisible"
      width="40%">
      <el-form :rules="rules" ref="createForm" :model="editForm" label-width="80px">
        <el-form-item label="用户名">
          <el-tag>{{editForm.username}}</el-tag>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button @click="edit" type="primary">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog
      title="分配角色"
      :visible.sync="assignVisible"
      width="40%">
      <el-form :rules="rules" ref="createForm" :model="assignForm" label-width="80px">
        <el-form-item label="用户名">
          <el-tag>{{assignForm.username}}</el-tag>
        </el-form-item>
        <el-form-item label="角色列表">
          <el-select v-model="assignForm.rid" placeholder="请选择">
            <el-option
              v-for="item in roles"
              :key="item.id"
              :label="item.roleName"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="assignVisible = false">取 消</el-button>
        <el-button @click="assign" type="primary">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
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
      total: 0,
      createVisible: false,
      editVisible: false,
      assignVisible: false,
      roles: [],
      form: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editForm: {
        id: 0,
        email: '',
        mobile: ''
      },
      assignForm: {
        id: 0,
        username: '',
        rid: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: ['change', 'blur'] },
          { min: 3, max: 12, message: '用户名长度在3到12个字符', trigger: ['change', 'blur'] }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: ['change', 'blur'] },
          { min: 3, max: 12, message: '密码长度在3到12个字符', trigger: ['change', 'blur'] }
        ],
        email: [
          { type: 'email', message: '请输入正确的邮箱地址', trigger: ['change', 'blur'] }
        ],
        mobile: [
          { pattern: /^1\d{10}$/, message: '请输入正确的手机号码', trigger: ['change', 'blur'] }
        ]

      }
    }
  },
  methods: {
    async getList () {
      const { meta, data } = await this.$axios.get('users', {
        params: {
          query: this.query,
          pagenum: this.pagenum,
          pagesize: this.pagesize
        }
      })
      if (meta.status === 200) {
        this.list = data.users
        this.total = data.total
      } else {
        this.$message.error(meta.msg)
      }
    },
    pageChange (page) {
      this.pagenum = page
      this.getList()
    },
    sizeChange (size) {
      this.pagesize = size
      this.pagenum = 1
      this.getList()
    },
    async stateChange (id, state) {
      try {
        const { meta } = await this.$axios.put(`users/${id}/state/${state}`)
        if (meta.status === 200) {
          this.$message.success('修改成功')
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    search () {
      this.getList()
    },
    async del (id) {
      try {
        await this.$confirm('确认要删除该用户吗?', '温馨提示 ')
        const { meta } = await this.$axios.delete(`users/${id}`)

        if (meta.status === 200) {
          if (Math.ceil((this.total - 1) / this.pagesize) < this.pagenum && this.pagenum > 2) {
            this.pagenum--
          }
          this.getList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    showCreate () {
      this.$refs.createForm && this.$refs.createForm.resetFields()
      this.createVisible = true
    },
    async create () {
      try {
        await this.$refs.createForm.validate()
        const { meta } = await this.$axios.post('users', this.form)
        if (meta.status === 201) {
          this.$message.success('添加用户成功')
          this.createVisible = false
          this.pagenum = Math.ceil((this.total + 1) / this.pagesize)
          this.getList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    showEdit (row) {
      this.editForm.id = row.id
      this.editForm.username = row.username
      this.editForm.email = row.email
      this.editForm.mobile = row.mobile
      this.editVisible = true
    },
    async edit () {
      const { id, email, mobile } = this.editForm
      const { meta } = await this.$axios.put(`users/${id}`, { email, mobile })
      if (meta.status === 200) {
        this.$message.success('修改成功')
        this.getList()
        this.editVisible = false
      } else {
        this.$message.success(meta.msg)
      }
    },
    async showAssign (row) {
      this.assignVisible = true
      this.assignForm.username = row.username
      this.assignForm.id = row.id

      const res = await this.$axios.get(`users/${row.id}`)
      if (res.meta.status === 200) {
        this.assignForm.rid = res.data.rid === -1 ? '' : res.data.rid
      } else {
        this.$message.error(res.meta.msg)
      }

      const { meta, data } = await this.$axios.get('roles')
      if (meta.status === 200) {
        this.roles = data
        console.log(data)
      } else {
        this.$message.error(meta.msg)
      }
    },
    async assign () {
      console.log(this.assignForm)
      const { rid, id } = this.assignForm
      if (this.assignForm.rid === '') {
        return this.$message.error('请选择角色')
      }
      const { meta } = await this.$axios.put(`users/${id}/role`, { rid })
      if (meta.status === 200) {
        this.$message.success('分配角色成功')
        this.assignVisible = false
      } else {
        this.$message.error(meta.msg)
      }
    }
  },
  watch: {

  }
}
</script>

<style lang="scss" scoped>
.users{

  .search-bar {
    .search-input {
      width:300px;
    }
    .create-btn {
      margin-left:10px;
    }
  }

  .el-table {
    margin-top:10px;
  }
  .el-pagination {
    margin-top:5px;
  }
}
</style>
