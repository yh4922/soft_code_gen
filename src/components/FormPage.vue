<template>
  <div>
    <el-row :gutter="15">
      <el-form ref="elForm" :model="formData" :rules="rules" size="medium" label-width="120px">
        <el-col :span="22">
          <el-form-item label="公司名称" prop="title">
            <el-input v-model="formData.title" placeholder="请输入手机号公司名称" show-word-limit clearable
              prefix-icon='el-icon-office-building' :style="{width: '100%'}"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="22">
          <el-form-item label="代码目录" prop="path">
            {{formData.path}}
            <el-button @click="openMulu" type="warning" icon="el-icon-folder-opened" size="mini"> 打开文件夹 </el-button>
          </el-form-item>
        </el-col>
        <el-col :span="22">
          <el-form-item label="代码文件" prop="extname">
            <el-input v-model="formData.extname" placeholder="代码文件的后缀代码文件" clearable :style="{width: '100%'}">
            </el-input>
          </el-form-item>
        </el-col>
        <el-col :span="22">
          <el-form-item size="large">
            <el-button type="primary" @click="submitForm">生成</el-button>
            <el-button @click="resetForm">重置</el-button>
          </el-form-item>
        </el-col>
      </el-form>
    </el-row>
  </div>
</template>
<script>
export default {
  components: {},
  props: [],
  data() {
    return {
      formData: {
        title: "深圳喜为智慧科技有限公司",
        path: '',
        // extname: ".cpp,.c,.h",
        extname: ".vue,.js,.ts,.css",
      },
      rules: {
        title: [{
          required: true,
          message: '请输入手机号公司名称',
          trigger: 'blur'
        }],
        path: [{
          required: true,
          message: '请选择代码所在目录'
        }],
        extname: [{
          required: true,
          message: '代码文件的后缀代码文件',
          trigger: 'blur'
        }],
      },
    }
  },
  computed: {},
  watch: {},
  created() {},
  mounted() {
  },
  methods: {
    openMulu () {
      this.tt.dialog.open({
        directory: true
      }).then(res => {
        this.formData.path = res
      })
    },
    submitForm() {
      this.$refs['elForm'].validate(valid => {
        if (!valid) return
        this.$emit('render', this.formData)
      })
    },
    resetForm() {
      this.$refs['elForm'].resetFields()
    },
  }
}

</script>
<style>
</style>
