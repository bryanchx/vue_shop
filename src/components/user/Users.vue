<template>
  <div>
    <!--      面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--      卡片试图区域-->
    <el-card>

      <el-row :gutter="20">
        <el-col :span="8">
          <!--     搜索与添加区域-->
          <el-input placeholder="请输入内容"
                    v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>
      <!--      用户列表区域-->
      <el-table :data="userList" border stripe>
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <!--            {{scope.row}}-->
            <el-switch
              v-model="scope.row.mg_state" @change="userStateChanged(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <!--            三个按钮-->
            <el-button type="primary" icon="el-icon-edit" size="mini"
                       @click="showEditDialog(scope.row.id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini"
                       @click="removeUserById(scope.row.id)"></el-button>
            <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!--      底部分页控件-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <!--    添加用户的对话框-->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed">
      <!--      内容主体区域-->
      <el-form :model="addForm" :rules="addFormRules"
               ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!--      底部区域-->
      <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addUser">确 定</el-button>
  </span>
    </el-dialog>
    <!--    修改用户对话框-->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed">
      <!--      内容主体-->
      <el-form :model="editForm" :rules="editFormRules"
               ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editUserInfo">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: 'Users',
    data () {
      //验证邮箱的规则
      var checkEmail = (rule, value, callback) => {
        const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
        if (regEmail.test(value)) {
          return callback()
        }
        callback(new Error('请输入合法邮箱'))
      }
      //验证邮手机号的规则
      var checkMobile = (rule, value, callback) => {
        const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
        if (regMobile.test(value)) {
          return callback()
        }
        callback(new Error('请输入合法手机号'))
      }
      return {
        //  获取用户列表的参数对象
        queryInfo: {
          query: '',
          pagenum: 1,
          pagesize: 2
        },
        userList: [],
        total: 0,
        addDialogVisible: false,
        //  添加用户表单数据
        addForm: {
          username: '',
          password: '',
          email: '',
          mobile: '',
        },
        //  查询到的用户表单数据
        editForm: {
          username: '',
          email: '',
          mobile: '',
        },
        //  添加表单验证规则对象
        addFormRules: {
          username: [
            {
              required: true,
              message: '请输入用户名',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 10,
              message: '长度在 3 到 10 个字符',
              trigger: 'blur'
            }
          ],
          password: [{
            required: true,
            message: '请输入登录密码',
            trigger: 'blur'
          },
            {
              min: 6,
              max: 15,
              message: '长度在 6 到 15 个字符',
              trigger: 'blur'
            }],
          email: [{
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          }, {
            validator: checkEmail,
            trigger: 'blur'
          }],
          mobile: [{
            required: true,
            message: '请输入手机号',
            trigger: 'blur'
          }, {
            validator: checkMobile,
            trigger: 'blur'
          }]
        },
        //  添加表单验证规则对象
        editFormRules: {
          email: [{
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          }, {
            validator: checkEmail,
            trigger: 'blur'
          }],
          mobile: [{
            required: true,
            message: '请输入手机号',
            trigger: 'blur'
          }, {
            validator: checkMobile,
            trigger: 'blur'
          }]
        },
        //控制修改用户对话框是否弹出
        editDialogVisible: false,
      }
    },
    created () {
      this.getUserList()
    }
    ,
    methods: {
      async getUserList () {
        const { data: res } = await this.$http.get('users', { params: this.queryInfo })
        if (res.meta.status !== 200) return this.$message.error('获取用户列表失败')
        this.userList = res.data.users
        this.total = res.data.total
        // console.log(this.userList)
      }
      ,
      //监听pagesize
      handleSizeChange (newSize) {
        // console.log(newSize)
        this.queryInfo.pagesize = newSize
        this.getUserList()
      }
      ,
      //监听页面改版
      handleCurrentChange (newPage) {
        // console.log(newPage)
        this.queryInfo.pagenum = newPage
        this.getUserList()
      }
      ,
      //监听switch状态开关
      async userStateChanged (userinfo) {
        const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
        if (res.meta.status !== 200) {
          userinfo.mg_state = !userinfo.mg_state
          return this.$message.error('更新用户状态失败')
        }
        this.$message.success('更新用户状态成功')
      },
      addDialogClosed () {
        this.$refs.addFormRef.resetFields()
      },
      editDialogClosed () {
        this.$refs.editFormRef.resetFields()
      },
      addUser () {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) {
            return
          }
          const { data: res } =
            await this.$http.post('users', this.addForm)
          if (res.meta.status !== 201) {
            this.$message.error('添加用户失败')
          }
          this.$message.success('添加用户成功')
          this.addDialogVisible = false
          this.getUserList()
        })
      },
      //修改用户信息并提交
      editUserInfo () {
        this.$refs.editFormRef.validate(async valid => {
          if (!valid) {
            return
          }
          const { data: res } =
            await this.$http.put('users/' + this.editForm.id,
              {
                email: this.editForm.email,
                mobile: this.editForm.mobile
              })
          if (res.meta.status !== 200) {
            this.$message.error('更新用户失败')
          }
          this.$message.success('更新用户成功')
          this.editDialogVisible = false
          this.getUserList()
        })
      },
      //编辑用户对话框
      async showEditDialog (id) {
        console.log(id)
        this.editDialogVisible = true
        const { data: res } = await this.$http.get('users/' + id)
        if (res.meta.status !== 200) {
          return this.$message.error('查询用户信息失败')
        }
        this.editForm = res.data
        this.editDialogVisible = true
      },
      async removeUserById (id) {
        //  弹框询问是否删除用户
        const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        //如果用户确认删除，返回值为confirm
        //如果用户取消删除，返回值为cancel
        // console.log(confirmResult)
        if (confirmResult !== 'confirm') {
          return this.$message.info('已取消删除')
        }
        const { data: res } = await this.$http.delete('users/' + id)
        if (res.meta.status!==200){
          return this.$message.error('删除失败')
        }
        this.$message.success('已删除')
        this.getUserList()
      }
    }
  }
</script>

<style scoped>

</style>
