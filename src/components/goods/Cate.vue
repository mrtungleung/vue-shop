<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCatDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table class="treeTable" :data="cateList" :columns="columns" :selection-type="false" :expand-type="false"
        show-index index-text="#" border :show-row-hover="false">
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen"></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCatDialogVisible"
      width="50%"
      @close="addCatDialogClosed"
      >
      <!-- 添加分类的表单 -->
      <el-form :model="addCatForm" :rules="addCatFormRules" ref="addCatFormRef" label-width="80px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCatForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <!-- options用来指定数据源 -->
          <!-- props用来指定配置对象 -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCatList"
            :props="cascaderProps"
            @change="parentCatChanged"
            clearable></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCatDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCat">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Cate',
  data () {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表
      cateList: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示将当前列定义为模板列
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          // 表示将当前列定义为模板列
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          // 表示将当前列定义为模板列
          type: 'template',
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCatDialogVisible: false,
      // 添加分类的表单数据对象
      addCatForm: {
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        cat_level: 0
      },
      // 添加分类表单的验证规则对象
      addCatFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      parentCatList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        checkStrictly: true,
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类的id数组
      selectedKeys: []
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败！')
      }
      console.log(res.data)
      // 把数据列表赋值给 cateList
      this.cateList = res.data.result
      // 为总数据条数赋值
      this.total = res.data.total
      // console.log(this.columns)
      console.log(this.cateList)
    },
    // 监听 pagesize 改变
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听 pagenum 改变
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮，显示添加分类对话框
    showAddCatDialog () {
      // 先获取父级分类的数据列表
      this.getParentCatList()
      // 再展示对话框
      this.addCatDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCatList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败！')
      }
      this.parentCatList = res.data
    },
    // 选择项发生变化触发这个函数
    parentCatChanged () {
      console.log(this.selectedKeys)
      // 如果selectedKeys的length > 0, 那么选中了父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类的id
        this.addCatForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 为当前分类的等级赋值
        this.addCatForm.cat_level = this.selectedKeys.length
      } else {
        this.addCatForm.cat_pid = 0
        this.addCatForm.cat_level = 0
      }
    },
    // 点击按钮，添加新的分类
    addCat () {
      this.$refs.addCatFormRef.validate(async valid => {
        if (!valid) {
          return false
        }
        const { data: res } = await this.$http.post('categories', this.addCatForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        await this.getCateList()
        this.addCatDialogVisible = false
      })
    },
    // 监听对话框的关闭事件，重置表单数据
    addCatDialogClosed () {
      this.$refs.addCatFormRef.resetFields()
      this.selectedKeys = []
      this.addCatForm.cat_level = 0
      this.addCatForm.cat_pid = 0
    }
  }
}
</script>

<style scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
