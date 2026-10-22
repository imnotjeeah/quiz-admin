<template>
  <div id="app">
    <el-container>
      <el-header style="background: #409EFF; color: white; display: flex; align-items: center; font-size: 20px;">
        Quiz 管理后台
      </el-header>
      <el-main>
        <!-- 搜索栏 -->
        <el-row :gutter="20" style="margin-bottom: 20px;">
          <el-col :span="8">
            <el-input v-model="searchKeyword" placeholder="搜索题目内容" clearable @clear="fetchData" />
          </el-col>
          <el-col :span="4">
            <el-button type="primary" @click="fetchData">搜索</el-button>
          </el-col>
          <el-col :span="4" :offset="8">
            <el-button type="success" @click="openDialog('add')">新增题目</el-button>
          </el-col>
        </el-row>

        <!-- 表格 -->
        <el-table :data="tableData" border stripe v-loading="loading">
          <el-table-column prop="id" label="ID" width="80" />
          <el-table-column prop="question" label="题目" min-width="200" show-overflow-tooltip />
          <el-table-column prop="optionA" label="选项A" width="120" />
          <el-table-column prop="optionB" label="选项B" width="120" />
          <el-table-column prop="optionC" label="选项C" width="120" />
          <el-table-column prop="answer" label="答案" width="80">
            <template slot-scope="scope">
              <el-tag type="success">{{ scope.row.answer }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="180" fixed="right">
            <template slot-scope="scope">
              <el-button size="mini" @click="openDialog('edit', scope.row)">编辑</el-button>
              <el-button size="mini" type="danger" @click="handleDelete(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>

        <!-- 分页 -->
        <el-pagination
          style="margin-top: 20px; text-align: right;"
          background
          layout="total, prev, pager, next"
          :total="filteredData.length"
          :page-size="pageSize"
          :current-page.sync="currentPage"
          @current-change="handlePageChange"
        />

        <!-- 新增/编辑弹窗 -->
        <el-dialog :title="dialogMode === 'add' ? '新增题目' : '编辑题目'" :visible.sync="dialogVisible" width="500px">
          <el-form :model="form" label-width="80px">
            <el-form-item label="题目">
              <el-input v-model="form.question" type="textarea" rows="3" />
            </el-form-item>
            <el-form-item label="选项A">
              <el-input v-model="form.optionA" />
            </el-form-item>
            <el-form-item label="选项B">
              <el-input v-model="form.optionB" />
            </el-form-item>
            <el-form-item label="选项C">
              <el-input v-model="form.optionC" />
            </el-form-item>
            <el-form-item label="答案">
              <el-select v-model="form.answer" style="width: 100%;">
                <el-option label="A" value="A" />
                <el-option label="B" value="B" />
                <el-option label="C" value="C" />
              </el-select>
            </el-form-item>
          </el-form>
          <span slot="footer">
            <el-button @click="dialogVisible = false">取消</el-button>
            <el-button type="primary" @click="saveForm">确定</el-button>
          </span>
        </el-dialog>
      </el-main>
    </el-container>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      searchKeyword: '',
      loading: false,
      currentPage: 1,
      pageSize: 5,
      dialogVisible: false,
      dialogMode: 'add',
      form: {
        id: null,
        question: '',
        optionA: '',
        optionB: '',
        optionC: '',
        answer: 'A'
      },
      // 模拟数据，部署后无需后端也能展示
      allData: [
        { id: 1, question: 'Vue 中哪个指令用于绑定事件？', optionA: 'v-bind', optionB: 'v-on', optionC: 'v-model', answer: 'B' },
        { id: 2, question: 'Element UI 的表格组件是？', optionA: 'el-form', optionB: 'el-table', optionC: 'el-dialog', answer: 'B' },
        { id: 3, question: 'Vue 实例中存放数据的是哪个选项？', optionA: 'methods', optionB: 'computed', optionC: 'data', answer: 'C' },
        { id: 4, question: 'HTTP 请求中 GET 方法通常用于？', optionA: '获取数据', optionB: '提交数据', optionC: '删除数据', answer: 'A' },
        { id: 5, question: 'CSS 中设置字体大小的属性是？', optionA: 'font-weight', optionB: 'font-size', optionC: 'font-family', answer: 'B' },
        { id: 6, question: 'JavaScript 中声明常量的关键字是？', optionA: 'var', optionB: 'let', optionC: 'const', answer: 'C' },
      ]
    }
  },
  computed: {
    filteredData() {
      if (!this.searchKeyword) return this.allData
      const kw = this.searchKeyword.toLowerCase()
      return this.allData.filter(item =>
        item.question.toLowerCase().includes(kw)
      )
    },
    tableData() {
      const start = (this.currentPage - 1) * this.pageSize
      return this.filteredData.slice(start, start + this.pageSize)
    }
  },
  mounted() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      this.loading = true
      setTimeout(() => {
        this.currentPage = 1
        this.loading = false
      }, 200)
    },
    handlePageChange(page) {
      this.currentPage = page
    },
    openDialog(mode, row) {
      this.dialogMode = mode
      if (mode === 'edit' && row) {
        this.form = { ...row }
      } else {
        this.form = { id: Date.now(), question: '', optionA: '', optionB: '', optionC: '', answer: 'A' }
      }
      this.dialogVisible = true
    },
    saveForm() {
      if (!this.form.question.trim()) {
        this.$message.warning('请输入题目内容')
        return
      }
      if (this.dialogMode === 'add') {
        this.allData.unshift({ ...this.form })
        this.$message.success('新增成功')
      } else {
        const index = this.allData.findIndex(item => item.id === this.form.id)
        if (index > -1) {
          this.allData.splice(index, 1, { ...this.form })
        }
        this.$message.success('修改成功')
      }
      this.dialogVisible = false
    },
    handleDelete(row) {
      this.$confirm('确定删除该题目吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const index = this.allData.findIndex(item => item.id === row.id)
        if (index > -1) this.allData.splice(index, 1)
        this.$message.success('删除成功')
      }).catch(() => {})
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  margin: 0;
  background: #f5f5f5;
  min-height: 100vh;
}
.el-header {
  line-height: 60px;
}
.el-main {
  background: white;
  margin: 20px;
  border-radius: 4px;
}
</style>