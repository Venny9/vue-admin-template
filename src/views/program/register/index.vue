<template>
  <div class="app-container">
    <el-form ref="form" :model="form" label-width="120px">
      <el-form-item label="项目英文名称">
        <el-input v-model="form.name" />
      </el-form-item>
      <el-form-item label="Git仓库地址">
        <el-input v-model="form.gitpath" />
      </el-form-item>
      <el-form-item label="项目代码路径">
        <el-input v-model="form.codepath" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit">Create</el-button>
        <el-button @click="onCancel">Cancel</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      form: {
        name: '',
        gitpath: '',
        codepath: '',
        fileList: []
      }
    }
  },
  methods: {
    onSubmit() {
      this.$message(this.form.name)
    },
    onCancel() {
      this.$message({
        message: 'cancel!',
        type: 'warning'
      })
    }
  }
}
</script>

<style scoped>
.line {
  text-align: center;
}
</style>
