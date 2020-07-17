<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert
        show-icon
        closable
        title="注意：只允许为第三级分类设置相关参数"
        type="warning">
      </el-alert>

      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col :span="5">
          <span>选择商品分类：</span>
        </el-col>
        <el-col :span="8">
          <el-cascader
            v-model="selectCateKeys"
            :options="cateList"
            :props="{ expandTrigger: 'hover' }"
            @change="handleChange"
          >
          </el-cascader>
        </el-col>
      </el-row>

      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" :disabled="isBtnDisabled" @click="addAttr">添加参数</el-button>
          <!-- 动态表格 -->
          <el-table :data="manyTableData" border striper>
            <el-table-column type="expand"></el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="editAttr(scope.row.attr_id)">编辑</el-button>
                <el-button size="mini"  type="warning" icon="el-icon-delete" @click="delAttr(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" :disabled="isBtnDisabled" @click="addAttr">添加属性</el-button>
           <!-- 静态表格 -->
          <el-table :data="onlyTableData" border striper>
            <el-table-column type="expand"></el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="editAttr(scope.row.attr_id)">编辑</el-button>
                <el-button size="mini" type="warning" icon="el-icon-delete" @click="delAttr(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加属性对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
      >
      <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="80px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

        <!-- 修改属性对话框 -->
    <el-dialog
      :title="'编辑' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
      >
      <el-form ref="editFormRef" :model="editForm" :rules="editFormRules" label-width="80px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>
<script>
export default {
  data () {
    return {
      cateList: [],
      selectCateKeys: [],
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      addDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      editForm: {
        attr_name: ''
      },
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getParentCateList()
  },
  methods: {
    // 获取父级分类的列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类失败')
      }
      console.log(res.data)
      // 处理渲染
      const news = res.data.map((item) => {
        return {
          value: item.cat_id,
          label: item.cat_name,
          children: item.children.map((item1) => {
            console.log(item1.children)
            return {
              value: item1.cat_id,
              label: item1.cat_name,
              // 注意此处必须 要有个默认空数组
              children: (item1.children || []).map((item2) => {
                return {
                  value: item2.cat_id,
                  label: item2.cat_name
                }
              })
            }
          })
        }
      })
      console.log(news)
      this.cateList = news
    },
    handleChange () {
      this.getParamsData()
    },
    handleTabClick () {
      console.log(this.activeName)
      this.getParamsData()
    },
    async getParamsData () {
      if (this.selectCateKeys.length !== 3) {
        this.selectCateKeys = []
        return
      }
      console.log(this.selectCateKeys)
      // 根据所选分类id和 当前所处的面板，获取对应的参数
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类失败')
      }
      console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    addAttr () {
      this.addDialogVisible = true
    },
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败')
        }
        this.$message.success('参加参数成功!')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    // 展示修改对话框
    async editAttr (attrId) {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attrId}`, {
        params: { sel: this.activeName }
      })
      console.log(res.data)
      if (res.meta.status !== 200) {
        return this.$message.error('添加参数失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        console.log(valid)
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败')
        }
        this.$message.success('修改成功！')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    async delAttr (attrId) {
      const comfirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (comfirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }

      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attrId}`, {
        attr_name: this.editForm.attr_name,
        attr_sel: this.activeName
      })
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败')
      }
      this.$message.success('删除成功！')
      this.getParamsData()
    }
  },
  computed: {
    isBtnDisabled () {
      if (this.selectCateKeys.length !== 3) {
        return true
      } else {
        return false
      }
    },
    cateId () {
      if (this.selectCateKeys.length === 3) {
        return this.selectCateKeys[2]
      }
      return null
    },
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>
<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
</style>
