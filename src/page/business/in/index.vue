<template>
  <Container>
    <el-form :model="form" :rules="rules" ref="form">
      <el-form-item label="日期" prop="num">
        <el-date-picker
          v-model="form.date"
          type="datetime"
          placeholder="选择日期"
          :clearable="false"
          :picker-options="pickerOptions">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="物品" prop="project">
        <ProjectSelect v-model="form.project"></ProjectSelect>
      </el-form-item>
      <el-form-item label="数量" prop="num">
        <el-input-number v-model="form.num" :step="1"></el-input-number>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="handleSubmit('form')">保存</el-button>
      </el-form-item>
    </el-form>
  </Container>
</template>

<script>
import vuex from '@/mixins/vuex.js'
export default {
  mixins: [
    vuex
  ],
  data () {
    return {
      form: {
        date: new Date(),
        project: null,
        num: 1
      },
      rules: {
        date: [
          { required: true, message: '请输入日期', trigger: 'blur' }
        ],
        project: [
          { required: true, message: '请输入物品名称', trigger: 'blur' }
        ],
        num: [
          { required: true, message: '请输入物品数量', trigger: 'change' },
          { type: 'number', min: 1, message: '物品数量最小为1', trigger: 'change' }
        ]
      },
      pickerOptions: {
        disabledDate: time => time.getTime() > Date.now(),
        shortcuts: [
          {
            text: '今天',
            onClick (picker) {
              picker.$emit('pick', new Date())
            }
          },
          {
            text: '昨天',
            onClick (picker) {
              const date = new Date()
              date.setTime(date.getTime() - 3600 * 1000 * 24)
              picker.$emit('pick', date)
            }
          }
        ]
      }
    }
  },
  methods: {
    handleSubmit (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          // 更新存量
          this.vuexProjectsUpdateNum({
            id: this.form.project,
            change: this.form.num
          })
          this.vuexProjectsLoad()
          // 历史
          this.vuexHistoryInPush({
            ...this.form,
            date: String(this.form.date),
            creatDate: String(new Date())
          })
          this.vuexHistoryInLoad()
          // 结束
          this.form.project = null
          this.form.num = 1
          this.$message({
            message: '保存成功',
            type: 'success'
          })
        } else {
          return false
        }
      })
    }
  }
}
</script>
