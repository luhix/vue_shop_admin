<template>
  <div>
        <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <el-table
        :data="cateList"
        style="width: 100%;margin-bottom: 20px;"
        row-key="cat_id"
        border
        :tree-props="{children: 'children', hasChildren: 'hasChildren'}">
        <el-table-column
          type="index"
          width="50">
        </el-table-column>
        <el-table-column
          prop="cat_name"
          label="分类名称">
        </el-table-column>
        <el-table-column
          prop="cat_deleted"
          label="是否有效">
          <template slot-scope="scope">
            <i v-if="scope.row.cat_deleted === false" style="color: red" class="el-icon-success"></i>
            <i v-else class="el-icon-error"></i>
          </template>
        </el-table-column>
        <el-table-column label="排序" prop="cat_level">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
            <el-tag type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
            <el-tag type="info" v-else>三级</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="操作">
          <template>
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 3, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
      >
      <el-form :model="addCateForm" :rules="addCateRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="props"
            @change="parentCateChange"
            clearable
          >
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>
<script>
export default {
  data () {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      cateList: [],
      total: 0,
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      parentCateList: [],
      props: {
        expandTrigger: 'hover',
        checkStrictly: true
      },
      selectedKeys: [] // 选中的id
    }
  },
  created () {
    this.getGoodsCateList()
  },

  methods: {
    async getGoodsCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      console.log(res)
      this.cateList = res.data.result
      this.total = res.data.total
    },

    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsCateList()
    },
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getGoodsCateList()
    },
    showAddCateDialog () {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类的列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类失败')
      }
      // 处理渲染
      const news = res.data.map((item) => {
        return {
          value: item.cat_id,
          label: item.cat_name,
          children: item.children.map((item) => {
            return {
              value: item.cat_id,
              label: item.cat_name
            }
          })
        }
      })
      console.log(news)
      this.parentCateList = news
    },
    parentCateChange (value) {
      console.log(this.selectedKeys)
      // 数组长度大于0， 选中父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 添加分类
    addCate () {
      this.$refs.addCateFormRef.validate(async (valid) => {
        console.log(valid)
        if (valid) {
          const { data: res } = await this.$http.post('categories', this.addCateForm)
          if (res.meta.status !== 201) {
            return this.$message.error('获取商品分类失败')
          }
          this.$message.success('获取商品分类成功')
          this.getGoodsCateList()
          this.addCateDialogVisible = false
        }
      })
    },
    // 监听对话框的关闭事件，重置表单数据
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    }
  }
}
</script>
<style lang="less" scoped>
.el-table {
  margin-top: 15px;
}
.el-cascader-panel {
  height: 200px !important;
}
</style>
