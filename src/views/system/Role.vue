/**
 * 系统管理  角色管理
 */
<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>角色管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索筛选 -->
    <el-form :inline="true" :model="formInline" class="user-search">

      <el-form-item label="搜索：">
        <el-input size="small" v-model="formInline.code" placeholder="输入角色代码"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button size="small" type="primary" icon="el-icon-search" @click="search">搜索</el-button>
        <el-button size="small" type="primary" icon="el-icon-plus" @click="handleEdit()">添加</el-button>
      </el-form-item>
    </el-form>
    <!--列表-->
    <el-table size="small" :data="listData" highlight-current-row v-loading="loading" border element-loading-text="拼命加载中" style="width: 100%;">
      <el-table-column align="center" prop="name" label="角色名称" width="150">
      </el-table-column>
      <el-table-column align="center" prop="code" label="角色代码" width="150">
      </el-table-column>
      <el-table-column align="center" prop="permissionList" label="权限列表" min-width="300" show-overflow-tooltip>
      </el-table-column>
      <el-table-column align="center" prop="createTime" label="创建时间" min-width="150">
      </el-table-column>
      <el-table-column align="center" prop="creator" label="创建人" width="150">
      </el-table-column>
      <el-table-column align="center" prop="updateTime" label="更新时间" min-width="150">
      </el-table-column>
      <el-table-column align="center" prop="modifier" label="修改人" width="150">
      </el-table-column>
      <el-table-column align="center" label="操作" min-width="150">
        <template slot-scope="scope">
          <!-- <el-button size="mini" @click="handleEdit(scope.$index, scope.row)">编辑</el-button> -->
          <!-- <el-button size="mini" type="danger" @click="deleteUser(scope.$index, scope.row)">删除</el-button> -->
          <el-button size="mini" type="success" @click="editPermission(scope.$index, scope.row)">权限编辑</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页组件 -->
    <Pagination v-bind:child-msg="pageparm" @callFather="callFather"></Pagination>

    <!-- 编辑界面 -->
    <el-dialog title="添加角色" :visible.sync="editFormVisible" width="30%" @click="closeDialog">
      <el-form label-width="120px" :model="editForm" :rules="rules" ref="editForm">
        <el-form-item label="角色名称" prop="name">
          <el-input size="small" v-model="editForm.name" auto-complete="off" placeholder="权限名称"></el-input>
        </el-form-item>
        <el-form-item label="角色CODE" prop="code">
          <el-input size="small" v-model="editForm.code" auto-complete="off" placeholder="权限CODE"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <!-- <el-button size="small" @click="closeDialog">取消</el-button> -->
        <el-button size="small" type="primary" :loading="loading" class="title" @click="submitForm('editForm')">保存</el-button>
      </div>
    </el-dialog>

    <!-- 编辑界面 -->
        <el-dialog title="权限编辑" :visible.sync="editPermissionFormVisible" width="30%" @click='closeDialog("edit")'>
      <el-form label-width="120px" :model="editPermissionForm" ref="editPermissionForm" :rules="rules">
        <el-form-item label="权限" prop="codeList">
          <el-select v-model="editPermissionForm.codeList" multiple placeholder="请选择">
            <el-option
              v-for="item in permissionList"
              :key="item.code"
              :label="item.name"
              :value="item.code">
            </el-option>
          </el-select>

        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <!-- <el-button size="small" @click='closeDialog("edit")'>取消</el-button> -->
        <el-button size="small" type="primary" :loading="loading" class="title" @click="submitEditPermissionForm('editPermissionForm')">保存</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  roleList,
  roleSave,
  permissionListAll,
  permissionAdd
} from '../../api/request'
import Pagination from '../../components/Pagination'
export default {
  data() {
    return {
      nshow: true, //switch开启
      fshow: false, //switch关闭
      loading: false, //是显示加载
      editFormVisible: false,
      editPermissionFormVisible: false, 
      title: '添加',
      editForm: {
        name: '',
        code: '',
      },
      addRoleForm: {
        uid: '',
      },
      // rules 表单验证
      rules: {
        code: [
          { required: true, message: '请输入角色代码', trigger: 'blur' }
        ],
        name: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ]
      },
      formInline: {
        page: 1,
        limit: 10,
        code: '',
      },
      // 删除
      seletedata: {
        ids: '',
        token: localStorage.getItem('logintoken')
      },
      userparm: [], //搜索权限
      listData: [], //用户数据
      // 数据权限
      RoleRight: [],
      RoleRightProps: {
        children: 'children',
        label: 'name'
      },
      // 选中
      checkmenu: [],
      //参数role
      saveroleId: '',
      // 分页参数
      pageparm: {
        currentPage: 1,
        pageSize: 10,
        total: 10
      },

      permissionList: [],
      editPermissionForm: {
        codeList: [],
        roleId:''
      }
    }
  },
  // 注册组件
  components: {
    Pagination
  },
  /**
   * 数据发生改变
   */

  watch: {},

  /**
   * 创建完毕
   */

  created() {
    this.getdata(this.formInline)
  },

  /**
   * 里面的方法只有被调用才会执行
   */

  methods: {
    // 获取角色列表
    getdata(parameter) {
      /***
       * 调用接口，注释上面模拟数据 取消下面注释
       */
      roleList(parameter)
        .then(res => {
          this.loading = false
          if (res.code != 0) {
            this.$message({
              type: 'info',
              message: res.msg
            })
          } else {
            this.listData = res.data.dataList
            // 分页赋值
            this.pageparm.currentPage = this.formInline.page
            this.pageparm.pageSize = this.formInline.limit
            this.pageparm.total = res.data.total
          }
        })
        .catch(err => {
          this.loading = false
          this.$message.error('获取角色列表失败，请稍后再试！')
        })
    },
    // 分页插件事件
    callFather(parm) {
      this.formInline.page = parm.currentPage
      this.formInline.limit = parm.pageSize
      this.getdata(this.formInline)
    },
    // 搜索事件
    search() {
      this.getdata(this.formInline)
    },
    //显示编辑界面
    handleEdit: function(index, row) {
      this.editFormVisible = true
    },
    // 编辑、增加页面保存方法
    submitForm(editData) {
      this.$refs[editData].validate(valid => {
        if (valid) {
          roleSave(this.editForm)
            .then(res => {
              this.editFormVisible = false
              this.loading = false
              if (res.code == 0) {
                this.getdata(this.formInline)
                this.$message({
                  type: 'success',
                  message: '角色保存成功！'
                })
              } else {
                this.$message({
                  type: 'info',
                  message: res.msg
                })
              }
              this.editForm = {}
            })
            .catch(err => {
              this.editForm = {}
              this.editFormVisible = false
              this.loading = false
              this.$message.error('角色保存失败，请稍后再试！')
            })
        } else {
          return false
        }
      })
    },
    // 删除角色
    deleteUser(index, row) {
      this.$confirm('确定要删除吗?', '信息', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          roleDelete(row.roleId)
            .then(res => {
              if (res.success) {
                this.$message({
                  type: 'success',
                  message: '角色已删除！'
                })
                this.getdata(this.formInline)
              } else {
                this.$message({
                  type: 'info',
                  message: res.msg
                })
              }
            })
            .catch(err => {
              this.loading = false
              this.$message.error('角色删除失败，请稍后再试！')
            })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 数据权限
    menuAccess: function(index, row) {
      this.menuAccessshow = true
      this.saveroleId = row.roleId
      RoleRightTree(row.roleId)
        .then(res => {
          if (res.data.success) {
            this.$message({
              type: 'success',
              message: '获取权限成功'
            })
            this.changemenu(res.data.data)
            this.RoleRight = this.changeArr(res.data.data)
          } else {
            this.$message({
              type: 'info',
              message: res.data.msg
            })
          }
        })
        .catch(err => {
          this.loading = false
          this.$message.error('获取权限失败，请稍后再试！')
        })
    },

    handleRole: function(index, row){
      this.addRoleFormVisible = true;


    },
    submitAddRoleForm: function(addRole){
        console.log(addRole)
    },
    // 选中菜单
    changemenu(arr) {
      let change = []
      for (let i = 0; i < arr.length; i++) {
        if (arr[i].checked) {
          change.push(arr[i].id)
        }
      }
      this.checkmenu = change
    },
    // tree 递归
    changeArr(list) {
      var temptree = [],
        tree = [],
        items = []
      for (var i in list) {
        if (!temptree[list[i].id]) {
          var trow = {
            id: list[i].id,
            pId: list[i].pId,
            name: list[i].name,
            open: list[i].open,
            checked: list[i].checked,
            children: []
          }
          temptree[list[i].id] = trow
          items.push(trow)
        }
        if (list[i].uid > 0) {
          temptree[list[i].id]['children'].push({
            id: list[i].id,
            pId: list[i].pId,
            name: list[i].name,
            open: list[i].open,
            checked: list[i].checked,
            children: []
          })
        }
      }

      for (var j in items) {
        if (temptree[items[j].pId]) {
          temptree[items[j].pId]['children'].push(temptree[items[j].id])
        } else {
          tree.push(temptree[items[j].id])
        }
      }
      temptree = null
      items = null
      return tree
    },
    // 菜单权限-保存
    menuPermSave() {
      let parm = {
        roleId: this.saveroleId,
        moduleIds: ''
      }
      let node = this.$refs.tree.getCheckedNodes()
      let moduleIds = []
      if (node.length != 0) {
        for (let i = 0; i < node.length; i++) {
          moduleIds.push(node[i].id)
        }
        parm.moduleIds = JSON.stringify(moduleIds)
      }
      RoleRightSave(parm)
        .then(res => {
          if (res.success) {
            this.$message({
              type: 'success',
              message: '权限保存成功'
            })
            this.menuAccessshow = false
            this.getdata(this.formInline)
          } else {
            this.$message({
              type: 'info',
              message: res.msg
            })
          }
        })
        .catch(err => {
          this.loading = false
          this.$message.error('权限保存失败，请稍后再试！')
        })
    },
    // 关闭编辑、增加弹出框
    closeDialog(dialog) {
      this.editPermissionFormVisible = false
    },  
    editPermission: function(index, row){
      this.editPermissionFormVisible = true
      this.editPermissionForm.roleId = row.id
      permissionListAll().then(res =>{
        if (res.code===0) {
          this.permissionList = res.data
        } else {
          this.$message({
            type: 'info',
            message: res.msg
          })
        }
      })
    },
    submitEditPermissionForm: function(editPermissionForm){
       permissionAdd(this.editPermissionForm)
      .then(res => {
        this.editPermissionFormVisible = false
        if (res.code===0) {
          this.getdata(this.formInline)
          this.$message({
            type: 'success',
            message: '数据保存成功！'
          })
        } else {
          this.$message({
            type: 'info',
            message: res.msg
          })
        }
        this.editPermissionForm = {};
      })
      .catch(err => {
        this.editPermissionFormVisible = false
        this.$message.error('保存失败，请稍后再试！')
        this.editPermissionForm = {};
      }) 
    }
  },
}
</script>

<style scoped>
.user-search {
  margin-top: 20px;
}
.userRole {
  width: 100%;
}
</style>

 