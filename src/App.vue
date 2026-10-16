<template>
  <div id="app">
    <header class="topbar">Quiz后台管理</header>

    <div class="workspace">
      <aside class="sidebar">
        <div class="system-title">
          <i class="el-icon-message"></i>
          <span>系统信息管理</span>
          <i class="el-icon-arrow-up arrow"></i>
        </div>
        <button
          v-for="item in menuItems"
          :key="item.key"
          class="menu-item"
          :class="{ active: activeMenu === item.key }"
          @click="switchMenu(item.key)"
        >
          {{ item.label }}
        </button>
      </aside>

      <main class="content">
        <section v-if="activeMenu === 'users'">
          <div class="toolbar">
            <label>用户名</label>
            <el-input
              v-model.trim="userKeyword"
              placeholder="用户名"
              clearable
              @keyup.enter.native="searchUsers"
            ></el-input>
            <el-button type="primary" @click="searchUsers">查询用户</el-button>
            <el-button type="success" @click="openUserDialog()">添加用户</el-button>
          </div>

          <el-table :data="pagedUsers" style="width: 100%" empty-text="暂无用户">
            <el-table-column prop="date" label="日期" min-width="180">
              <template slot-scope="scope">
                <i class="el-icon-time date-icon"></i>{{ scope.row.date }}
              </template>
            </el-table-column>
            <el-table-column prop="username" label="用户名" min-width="170">
              <template slot-scope="scope">
                <span class="name-tag">{{ scope.row.username }}</span>
              </template>
            </el-table-column>
            <el-table-column prop="password" label="密码" min-width="150"></el-table-column>
            <el-table-column label="操作" min-width="210">
              <template slot-scope="scope">
                <el-button size="mini" @click="openUserDialog(scope.row)">编辑</el-button>
                <el-button size="mini" type="danger" @click="removeUser(scope.row.id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>

          <el-pagination
            background
            layout="prev, pager, next"
            :page-size="pageSize"
            :total="filteredUsers.length"
            :current-page.sync="userPage"
          ></el-pagination>
        </section>

        <section v-else>
          <div class="toolbar question-toolbar">
            <label>题目</label>
            <el-input
              v-model.trim="questionKeyword"
              placeholder="请输入题目"
              clearable
              @keyup.enter.native="searchQuestions"
            ></el-input>
            <el-button type="success" @click="searchQuestions">查询题目</el-button>
            <el-button type="success" @click="openQuestionDialog()">添加题目</el-button>
          </div>

          <el-table :data="pagedQuestions" style="width: 100%" empty-text="暂无题目">
            <el-table-column prop="id" label="序号" width="120"></el-table-column>
            <el-table-column prop="title" label="问题" min-width="340"></el-table-column>
            <el-table-column prop="answer" label="答案" width="120"></el-table-column>
            <el-table-column label="操作" width="210">
              <template slot-scope="scope">
                <el-button size="mini" @click="openQuestionDialog(scope.row)">编辑</el-button>
                <el-button size="mini" type="danger" @click="removeQuestion(scope.row.id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>

          <el-pagination
            background
            layout="prev, pager, next"
            :page-size="pageSize"
            :total="filteredQuestions.length"
            :current-page.sync="questionPage"
          ></el-pagination>
        </section>
      </main>
    </div>

    <el-dialog
      :title="editingUserId ? '编辑用户' : '添加用户'"
      :visible.sync="userDialogVisible"
      width="500px"
      @closed="resetUserForm"
    >
      <el-form ref="userForm" :model="userForm" :rules="userRules" label-width="80px">
        <el-form-item label="日期" prop="date">
          <el-date-picker v-model="userForm.date" type="date" value-format="yyyy-MM-dd" placeholder="选择日期"></el-date-picker>
        </el-form-item>
        <el-form-item label="用户名" prop="username">
          <el-input v-model.trim="userForm.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model.trim="userForm.password" placeholder="请输入密码"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="userDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="saveUser">确定</el-button>
      </span>
    </el-dialog>

    <el-dialog
      title="题目"
      :visible.sync="questionDialogVisible"
      width="700px"
      @closed="resetQuestionForm"
    >
      <el-form ref="questionForm" :model="questionForm" :rules="questionRules" label-width="85px">
        <el-form-item label="问题内容" prop="title">
          <el-input v-model.trim="questionForm.title" placeholder="请输入问题内容"></el-input>
        </el-form-item>
        <el-form-item label="选项A" prop="optionA"><el-input v-model.trim="questionForm.optionA"></el-input></el-form-item>
        <el-form-item label="选项B" prop="optionB"><el-input v-model.trim="questionForm.optionB"></el-input></el-form-item>
        <el-form-item label="选项C" prop="optionC"><el-input v-model.trim="questionForm.optionC"></el-input></el-form-item>
        <el-form-item label="选项D" prop="optionD"><el-input v-model.trim="questionForm.optionD"></el-input></el-form-item>
        <el-form-item label="答案" prop="answer">
          <el-select v-model="questionForm.answer" placeholder="请选择答案">
            <el-option v-for="option in ['A', 'B', 'C', 'D']" :key="option" :label="option" :value="option"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="questionDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="saveQuestion">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
const today = () => new Date().toISOString().slice(0, 10)

const seedUsers = [
  { id: 1, date: '2016-05-02', username: '王小虎', password: '123456' },
  { id: 2, date: '2016-05-04', username: '王小虎', password: '123456' },
  { id: 3, date: '2016-05-01', username: '王小虎', password: '123456' },
  { id: 4, date: '2016-05-03', username: '王小虎', password: '123456' },
  { id: 5, date: '2016-05-06', username: '李小明', password: '654321' },
  { id: 6, date: '2016-05-08', username: '赵小雨', password: '123456' }
]

const seedQuestions = [
  { id: 1, title: 'What does HTML stand for?', optionA: 'HyperText Markup Language', optionB: 'HighText Machine Language', optionC: 'Hyperlink and Text Markup Language', optionD: 'Home Tool Markup Language', answer: 'A' },
  { id: 2, title: 'Which language runs directly in a web browser?', optionA: 'Python', optionB: 'Java', optionC: 'JavaScript', optionD: 'C++', answer: 'C' },
  { id: 3, title: 'What does CSS stand for?', optionA: 'Creative Style Sheets', optionB: 'Cascading Style Sheets', optionC: 'Computer Style Sheets', optionD: 'Colorful Style Sheets', answer: 'B' },
  { id: 4, title: 'Which HTML tag is used to create a hyperlink?', optionA: '<link>', optionB: '<href>', optionC: '<url>', optionD: '<a>', answer: 'D' },
  { id: 5, title: 'Which CSS property changes text color?', optionA: 'font-color', optionB: 'text-color', optionC: 'color', optionD: 'background-color', answer: 'C' },
  { id: 6, title: 'What does JS stand for?', optionA: 'JavaSystem', optionB: 'JavaScript', optionC: 'JustScript', optionD: 'JScript', answer: 'B' },
  { id: 7, title: 'Which symbol is used for comments in JavaScript?', optionA: '<!-- -->', optionB: '//', optionC: '##', optionD: '/* */', answer: 'B' },
  { id: 8, title: "What does 'www' stand for?", optionA: 'World Wide Web', optionB: 'Web World Wide', optionC: 'Wide World Web', optionD: 'World Web Wide', answer: 'A' }
]

function loadData (key, fallback) {
  try {
    const data = JSON.parse(localStorage.getItem(key))
    return Array.isArray(data) ? data : fallback
  } catch (error) {
    return fallback
  }
}

export default {
  name: 'App',
  data () {
    return {
      activeMenu: 'users',
      menuItems: [
        { key: 'questions', label: '题目管理' },
        { key: 'users', label: '用户管理' }
      ],
      users: loadData('quiz-admin-users', seedUsers),
      questions: loadData('quiz-admin-questions', seedQuestions),
      userKeyword: '',
      appliedUserKeyword: '',
      questionKeyword: '',
      appliedQuestionKeyword: '',
      pageSize: 4,
      userPage: 1,
      questionPage: 1,
      userDialogVisible: false,
      questionDialogVisible: false,
      editingUserId: null,
      editingQuestionId: null,
      userForm: { date: today(), username: '', password: '' },
      questionForm: { title: '', optionA: '', optionB: '', optionC: '', optionD: '', answer: '' },
      userRules: {
        date: [{ required: true, message: '请选择日期', trigger: 'change' }],
        username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }]
      },
      questionRules: {
        title: [{ required: true, message: '请输入问题内容', trigger: 'blur' }],
        optionA: [{ required: true, message: '请输入选项A', trigger: 'blur' }],
        optionB: [{ required: true, message: '请输入选项B', trigger: 'blur' }],
        optionC: [{ required: true, message: '请输入选项C', trigger: 'blur' }],
        optionD: [{ required: true, message: '请输入选项D', trigger: 'blur' }],
        answer: [{ required: true, message: '请选择答案', trigger: 'change' }]
      }
    }
  },
  computed: {
    filteredUsers () {
      const keyword = this.appliedUserKeyword.toLowerCase()
      return this.users.filter(item => item.username.toLowerCase().includes(keyword))
    },
    pagedUsers () {
      const start = (this.userPage - 1) * this.pageSize
      return this.filteredUsers.slice(start, start + this.pageSize)
    },
    filteredQuestions () {
      const keyword = this.appliedQuestionKeyword.toLowerCase()
      return this.questions.filter(item => item.title.toLowerCase().includes(keyword))
    },
    pagedQuestions () {
      const start = (this.questionPage - 1) * this.pageSize
      return this.filteredQuestions.slice(start, start + this.pageSize)
    }
  },
  methods: {
    persist () {
      localStorage.setItem('quiz-admin-users', JSON.stringify(this.users))
      localStorage.setItem('quiz-admin-questions', JSON.stringify(this.questions))
    },
    switchMenu (key) {
      this.activeMenu = key
    },
    searchUsers () {
      this.appliedUserKeyword = this.userKeyword
      this.userPage = 1
    },
    searchQuestions () {
      this.appliedQuestionKeyword = this.questionKeyword
      this.questionPage = 1
    },
    openUserDialog (row) {
      this.editingUserId = row ? row.id : null
      this.userForm = row ? { date: row.date, username: row.username, password: row.password } : { date: today(), username: '', password: '' }
      this.userDialogVisible = true
    },
    saveUser () {
      this.$refs.userForm.validate(valid => {
        if (!valid) return
        if (this.editingUserId) {
          const index = this.users.findIndex(item => item.id === this.editingUserId)
          this.$set(this.users, index, { id: this.editingUserId, ...this.userForm })
        } else {
          this.users.push({ id: Date.now(), ...this.userForm })
        }
        this.persist()
        this.userDialogVisible = false
        this.$message.success(this.editingUserId ? '用户修改成功' : '用户添加成功')
      })
    },
    removeUser (id) {
      this.$confirm('确定要删除该用户吗？', '提示', { type: 'warning' }).then(() => {
        this.users = this.users.filter(item => item.id !== id)
        this.persist()
        this.$message.success('删除成功')
      }).catch(() => {})
    },
    resetUserForm () {
      this.editingUserId = null
      if (this.$refs.userForm) this.$refs.userForm.resetFields()
    },
    openQuestionDialog (row) {
      this.editingQuestionId = row ? row.id : null
      this.questionForm = row
        ? { title: row.title, optionA: row.optionA, optionB: row.optionB, optionC: row.optionC, optionD: row.optionD, answer: row.answer }
        : { title: '', optionA: '', optionB: '', optionC: '', optionD: '', answer: '' }
      this.questionDialogVisible = true
    },
    saveQuestion () {
      this.$refs.questionForm.validate(valid => {
        if (!valid) return
        if (this.editingQuestionId) {
          const index = this.questions.findIndex(item => item.id === this.editingQuestionId)
          this.$set(this.questions, index, { id: this.editingQuestionId, ...this.questionForm })
        } else {
          const nextId = this.questions.length ? Math.max(...this.questions.map(item => item.id)) + 1 : 1
          this.questions.push({ id: nextId, ...this.questionForm })
        }
        this.persist()
        this.questionDialogVisible = false
        this.$message.success(this.editingQuestionId ? '题目修改成功' : '题目添加成功')
      })
    },
    removeQuestion (id) {
      this.$confirm('确定要删除该题目吗？', '提示', { type: 'warning' }).then(() => {
        this.questions = this.questions.filter(item => item.id !== id)
        this.persist()
        this.$message.success('删除成功')
      }).catch(() => {})
    },
    resetQuestionForm () {
      this.editingQuestionId = null
      if (this.$refs.questionForm) this.$refs.questionForm.resetFields()
    }
  }
}
</script>

<style>
* { box-sizing: border-box; }
html, body, #app { min-height: 100%; margin: 0; }
body {
  color: #5d6470;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Microsoft YaHei", Arial, sans-serif;
  background: #fff;
}
.topbar {
  height: 66px;
  padding: 8px 20px;
  color: #1f2530;
  font-size: 38px;
  font-weight: 400;
  line-height: 50px;
  background: #eef2f7;
  border-bottom: 1px solid #e7ebf0;
}
.workspace { display: flex; min-height: calc(100vh - 66px); }
.sidebar {
  flex: 0 0 245px;
  min-height: calc(100vh - 66px);
  padding-top: 14px;
  background: #f7f9fc;
  border-right: 1px solid #edf0f4;
}
.system-title {
  display: flex;
  align-items: center;
  height: 42px;
  padding: 0 23px;
  color: #606772;
  font-size: 15px;
}
.system-title i:first-child { margin-right: 12px; }
.system-title .arrow { margin-left: auto; font-size: 12px; }
.menu-item {
  display: block;
  width: 100%;
  height: 52px;
  padding-left: 50px;
  color: #7765a4;
  font: inherit;
  text-align: left;
  text-decoration: underline;
  background: transparent;
  border: 0;
  cursor: pointer;
}
.menu-item:hover, .menu-item.active { background: #eaf3ff; color: #5d4b90; }
.content { flex: 1; padding: 22px 28px 60px; overflow: hidden; }
.toolbar { display: flex; align-items: center; gap: 14px; margin-bottom: 18px; }
.toolbar label { min-width: 46px; color: #606266; font-size: 14px; }
.toolbar .el-input { width: 200px; }
.question-toolbar .el-input { width: 260px; }
.date-icon { margin-right: 7px; color: #aeb6c2; }
.name-tag {
  display: inline-block;
  padding: 5px 10px;
  color: #409eff;
  background: #ecf5ff;
  border-radius: 3px;
}
.el-table { color: #68707b; }
.el-table th { color: #606772; font-weight: 500; }
.el-pagination { margin-top: 22px; }
.el-dialog__header { padding: 22px 24px; border-bottom: 1px solid #eef0f4; }
.el-dialog__body { padding: 30px 42px 12px; }
.el-dialog__footer { padding: 16px 24px 22px; }
.el-date-editor.el-input, .el-select { width: 100%; }

@media (max-width: 760px) {
  .topbar { height: 58px; font-size: 30px; line-height: 42px; }
  .workspace { display: block; }
  .sidebar { display: flex; min-height: auto; padding: 0; border-bottom: 1px solid #e5e8ee; }
  .system-title { display: none; }
  .menu-item { height: 48px; padding: 0; text-align: center; }
  .content { padding: 18px 12px 40px; }
  .toolbar { flex-wrap: wrap; gap: 8px; }
  .toolbar .el-input, .question-toolbar .el-input { flex: 1; min-width: 170px; width: auto; }
  .toolbar label { min-width: auto; }
  .el-dialog { width: calc(100% - 24px) !important; }
  .el-dialog__body { padding: 24px 14px 8px; }
}
</style>
