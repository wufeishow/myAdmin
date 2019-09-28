<template>
  <div class="roles">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-button @click="showAdd" type="success" plain>添加角色</el-button>

    <el-table :data="list">
        <el-table-column type="expand">
          <template v-slot:default="{row}">
          <span v-if="row.children.length === 0">没有权限</span>
          <el-row v-for="l1 in row.children" :key="l1.id">
            <el-col :span="4">
              <el-tag @close="del(row,l1.id)" closable>{{l1.authName}}</el-tag>
            </el-col>
            <el-col :span="20">
              <el-row v-for="l2 in l1.children" :key="l2.id">
                <el-col :span="4">
                  <el-tag @close="del(row,l2.id)" closable type="success">{{l2.authName}}</el-tag>
                </el-col>
                <el-col :span="20">
                  <el-tag @close="del(row,l3.id)" closable type="warning" v-for="l3 in l2.children" :key="l3.id">{{l3.authName}}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
          </template>
        </el-table-column>
      <el-table-column type="index"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作">
        <template v-slot:default="{row}">
          <el-button plain size="small" type="primary" @click="showEdit(row)" icon="el-icon-edit"></el-button>
          <el-button plain size="small" type="danger" @click="delRole(row.id)" icon="el-icon-delete"></el-button>
          <el-button plain size="small" type="success" @click="showAssign(row)" icon="el-icon-check">分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog
      title="提示"
      :visible.sync="assignVisible"
      width="30%">
      <el-tree
      ref="tree"
      :data="rights"
      default-expand-all
      show-checkbox
      :props="defaultProps"
      node-key="id"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="assignVisible = false">取 消</el-button>
        <el-button @click="assign" type="primary">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog
      :title="dialogTitle"
      :visible.sync="roleVisible"
      @close="dialogClose"
      width="40%">
      <el-form ref="roleForm" :model="form" :rules="rules" label-width="80px">
        <el-form-item  label="角色名称" prop="roleName">
          <el-input v-model="form.roleName"></el-input>
        </el-form-item>
        <el-form-item  label="角色描述" prop="roleDesc">
          <el-input v-model="form.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="roleVisible = false">取 消</el-button>
        <el-button @click="submitRole" type="primary">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      dialogTitle: '添加角色',
      list: [],
      assignVisible: false,
      roleVisible: false,
      rights: [],
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      checkeds: [],
      assignId: 0,
      form: {
        id: 0,
        roleName: '',
        roleDesc: ''
      },
      rules: {
        roleName: [
          { required: true, message: '角色名称不能为空', trigger: ['blur', 'change'] }
        ],
        roleDesc: [
          { required: true, message: '角色描述不能为空', trigger: ['blur', 'change'] }
        ]
      }
    }
  },
  created () {
    this.getList()
  },
  methods: {
    async getList () {
      try {
        const { meta, data } = await this.$axios.get('roles')
        if (meta.status === 200) {
          this.list = data
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    async del (row, rightId) {
      try {
        const { meta, data } = await this.$axios.delete(`roles/${row.id}/rights/${rightId}`)
        if (meta.status === 200) {
          row.children = data
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    async showAssign (row) {
      try {
        this.assignId = row.id
        this.assignVisible = true
        const { meta, data } = await this.$axios.get('rights/tree')
        if (meta.status === 200) {
          this.rights = data
        }
        this.$refs.tree.setCheckedKeys(this.getIds(row))
      } catch (e) {
        console.log(e)
      }
    },
    getIds (row) {
      let arr = []
      row.children.forEach(v => {
        if (v.children) {
          // arr = arr.concat(this.getIds(v))
          arr = [...arr, ...this.getIds(v)]
        } else {
          arr.push(v.id)
        }
      })
      return arr
    },
    async assign () {
      const checkeds = this.$refs.tree.getCheckedKeys()
      const halfs = this.$refs.tree.getHalfCheckedKeys()
      const rids = [...checkeds, ...halfs].join()
      const { meta } = await this.$axios.post(`roles/${this.assignId}/rights`, { rids })
      if (meta.status === 200) {
        this.$message.success('分配权限成功')
        this.assignVisible = false
        this.getList()
      }
    },
    async delRole (id) {
      try {
        await this.$confirm('确认要删除该角色吗?', '温馨提示')
        const { meta } = await this.$axios.delete(`roles/${id}`)
        if (meta.status === 200) {
          this.$message.success('删除成功')
          this.getList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    showAdd () {
      this.roleVisible = true
      this.dialogTitle = '添加角色'
    },
    showEdit (row) {
      this.roleVisible = true
      this.dialogTitle = '编辑角色'
      this.$nextTick(() => {
        this.form.roleName = row.roleName
        this.form.roleDesc = row.roleDesc
        this.form.id = row.id
      })
    },
    dialogClose () {
      this.$refs.roleForm.resetFields()
      this.form.id = 0
    },
    async submitRole () {
      const isAdd = this.dialogTitle === '添加角色'
      const url = isAdd ? 'roles' : `roles/${this.form.id}`
      const method = isAdd ? 'post' : 'put'
      const { meta } = await this.$axios[method](url, this.form)
      if (meta.status === 200 || meta.status === 201) {
        this.$message.success('操作成功')
        this.roleVisible = false
        this.getList()
      } else {
        this.$message.error(meta.msg)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  .roles {
    .el-table {
      margin-top:10px;
    }
    .el-row {
      margin:3px;
      .el-tag {
        margin:3px;
      }
    }
  }
</style>
