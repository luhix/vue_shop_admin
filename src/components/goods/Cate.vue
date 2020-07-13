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
          <el-button type="primary">添加分类</el-button>
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
      total: 0
      // 为table指定列的定义
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
    }
  }
}
</script>
<style lang="less" scoped>
.el-table {
  margin-top: 15px;
}
</style>
