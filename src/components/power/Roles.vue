<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/Home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
       </el-breadcrumb>
       <el-card :data="roleList">
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showAddRole()">添加角色</el-button>
                </el-col>
            </el-row>
            <el-table :data="roleList" border stripe>
                <el-table-column type="expand">
                  <template slot-scope="scope">
                    <el-row :class="['bdbottom','vcenter',i1 === 0 ?'bdtop' : '']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                      <el-col :span='5'>
                        <el-tag closable @close="removeRightById(scope.row, item1.id)">
                          {{item1.authName}}
                        </el-tag>
                         <i class="el-icon-caret-right">
                           </i>
                      </el-col>
                      <el-col :span='19'>
                          <el-row v-for="(item2, i2) in item1.children" :key="item2.id" :class="['vcenter',i2 === 0 ? '' :'bdtop']">
                          <el-col :span="6">
                              <el-tag type='success' closable @close="removeRightById(scope.row, item2.id)">
                                {{item2.authName}}
                              </el-tag>
                              <i class="el-icon-caret-right">
                           </i>
                          </el-col>
                          <el-col :span="18">
                            <el-tag v-for="(item3) in item2.children" :key="item3.id" type="warning" closable @close="removeRightById(scope.row, item3.id)">
                              {{item3.authName}}
                            </el-tag>
                          </el-col>
                        </el-row>
                      </el-col>
                    </el-row>
                  </template>
                </el-table-column>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName" width="330px"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc" width="330px"></el-table-column>
                <el-table-column label="操作" width="320px">
                    <template slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit"
                    size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
                    <el-button type="danger"  icon="el-icon-delete"
                    size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
                    <el-button type="warning"     size="mini"             icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
      </el-card>
      <el-dialog
            title="添加用户"
            :visible.sync="addRoleVisible"
            width="50%" @close="addDialogClosed()"
        >
            <el-form :model="addRole" :rules="addRoleRules" ref="addRoleRef" label-width="90px">
                <el-form-item label="角色名" prop="roleName">
                 <el-input v-model="addRole.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述" prop="roleDesc">
                 <el-input v-model="addRole.roleDesc"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                 <el-button @click="addRoleVisible = false">取 消</el-button>
                 <el-button type="primary" @click="addRoles()">确 定</el-button>
            </span>
        </el-dialog>
      <el-dialog
        title="修改角色"
        :visible.sync="editDialogVisible"
         width="50%" @close="editDialogClosed()">
         <el-form :model="editRole" :rules="editRoleRules" ref="editRoleRef" label-width="90px">
                <el-form-item label="角色名" prop="roleName">
                 <el-input v-model="editRole.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述" prop="roleDesc">
                 <el-input v-model="editRole.roleDesc"></el-input>
                </el-form-item>
         </el-form>
        <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible =  false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo()">确 定</el-button>
        </span>
        </el-dialog>
      <el-dialog
            title="分配权限"
            :visible.sync="setRightDialogVisble"
            width="50%" @close="setRightDialogClosed()"
        >
            <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" :default-expand-all="true" :default-checked-keys="defKeys" ref="treeRef"></el-tree>
            <span slot="footer" class="dialog-footer">
            <el-button @click="setRightDialogVisble = false">取 消</el-button>
            <el-button type="primary" @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
  data () {
    return {
      roleList: [],
      queryInfo: {
        roleName: '123',
        roleDesc: ''
      },
      addRole: {
        roleName: '工程师',
        roleDesc: ''
      },
      addRoleVisible: false,
      addRoleRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' },
          { min: 3, max: 10, message: '角色名的长度在3~10个字符之间', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 5, max: 12, message: '角色描述的长度在5~12个字符之间', trigger: 'blur' }
        ]
      },
      editRole: {},
      editDialogVisible: false,
      editRoleRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' },
          { min: 3, max: 10, message: '角色名的长度在3~10个字符之间', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 5, max: 12, message: '角色描述的长度在5~12个字符之间', trigger: 'blur' }
        ]
      },
      setRightDialogVisble: false,
      rightsList: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defKeys: [],
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles', { param: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.roleList = res.data
    },
    showAddRole () {
      this.addRoleVisible = true
    },
    addRoles () {
      this.$refs.addRoleRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRole)
        if (res.meta.status !== 201) {
          this.$message.error('添加角色失败')
        }
        this.$message.success('添加角色成功')
        this.addRoleVisible = false
        this.getRolesList()
      })
    },
    addDialogClosed () {
      this.$refs.addRoleRef.resetFields()
    },
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败')
      }
      this.editRole = res.data
      this.editDialogVisible = true
    },
    editDialogClosed () {
      this.$refs.editRoleRef.resetFields()
    },
    editUserInfo () {
      this.$refs.editRoleRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('roles/' + this.editRole.roleId, {
          roleName: this.editRole.roleName,
          roleDesc: this.editRole.roleDesc
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改角色数据失败')
        }
        this.editDialogVisible = false
        this.getRolesList()
        this.$message.success('修改角色数据成功')
      })
    },
    async removeRoleById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败')
      }
      this.$message.success('角色已删除')
      this.getRolesList()
    },
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.error('已取消删除！')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败！')
      }
      role.children = res.data
    },
    async showSetRightDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get(
        'rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限失败')
      }
      this.rightsList = res.data
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisble = true
    },
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    setRightDialogClosed () {
      this.defKeys = []
    },
    async allotRights () {
      const keys = [
        this.$refs.treeRef.getCheckedKeys(),
        this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }
      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisble = false
    }
  }
}
</script>
<style lang="less" scoped>
.el-tag{
    margin: 7px;
}
.bdtop{
    border-top:1px solid #eee
}
.bdbottom{
    border-bottom:1px solid #eee
}
</style>
