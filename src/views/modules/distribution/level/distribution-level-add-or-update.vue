<template>
  <el-dialog :title="!dataForm.id ? $t('brand.add') : $t('brand.revise')" :close-on-click-modal="false"
    :append-to-body="visible" :visible.sync="visible" width="40%">
    <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm"
      @keyup.enter.native="dataFormSubmit()" label-width="100px" size="small" class="body-content">
      <el-form-item label="等级名称" placeholder="等级名称" prop="name" class="input-width">
        <el-input v-model="dataForm.name" maxlength="30" show-word-limit></el-input>
      </el-form-item>
      <el-form-item label="分销比例" placeholder="分销比例" prop="gradeRatio" class="input-width">
        <el-input-number controls-position="right" v-model="dataForm.gradeRatio" :min="0" :max="100"></el-input-number>
        <el-tooltip class="item" effect="light" content="百分比,请输入 0-100 的整数,输入 1 即 1%。" placement="right" style="color: #155BD4;">
          <i class="el-icon-question" />
        </el-tooltip>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <div class="default-btn" @click="visible = false">{{
        $t("remindPop.cancel")
      }}</div>
      <div class="primary-btn default-btn" @click="dataFormSubmit()">{{
        $t("remindPop.confirm")
      }}</div>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      isSubmit: false,
      dataForm: {
        id: 0,
        name: '',
        gradeRatio: 0
      },
      dataRule: {
        name: [
          { required: true, message: '请输入等级名称', trigger: 'blur' },
        ]
      }
    }
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0
      this.visible = true
      this.isSubmit = false
      this.dataForm.name = null
      this.dataForm.gradeRatio = 0
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl('/platform/distributionConfig/user/info'),
            method: 'get',
            params: this.$http.adornParams(
              Object.assign({ id: this.dataForm.id, current: 1 }), false
            )
          }).then(({ data }) => {
            this.dataForm.name = data[0].name
            this.dataForm.gradeRatio = data[0].gradeRatio
          })
        }
      })
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.isSubmit) {
            return false
          }
          this.isSubmit = true
          if (this.dataForm.id) {
            console.log('post')
            this.$http({
              url: this.$http.adornUrl('/platform/distributionConfig/update/user/info'),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id,
                'name': this.dataForm.name,
                'gradeRatio': this.dataForm.gradeRatio
              })
            }).then(({ data }) => {
              this.$message({
                message: this.$i18n.t('remindPop.succeeded'),
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.visible = false
                  this.isSubmit = false
                  this.$emit('refreshDataList')
                }
              })
            }).catch(() => {
              this.isSubmit = false
            })
          } else {
            this.$http({
              url: this.$http.adornUrl('/platform/distributionConfig/add/user/info'),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id,
                'name': this.dataForm.name,
                'gradeRatio': this.dataForm.gradeRatio
              }, false)
            }).then(({ data }) => {
              this.$message({
                message: this.$i18n.t('remindPop.succeeded'),
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.visible = false
                  this.isSubmit = false
                  this.$emit('refreshDataList')
                }
              })
            }).catch(() => {
              this.isSubmit = false
            })
          }

        }
      })
    }
  }
}
</script>
<style lang="scss">
.body-content {
  overflow: auto;
}

el-form-item {
  margin-bottom: 80px;
}
</style>
