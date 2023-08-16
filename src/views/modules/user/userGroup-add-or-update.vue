<template>
  <el-dialog :title="!dataForm.id ? $t('brand.add') : $t('brand.revise')" :close-on-click-modal="false"
    :append-to-body="visible" :visible.sync="visible">
    <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm"
      @keyup.enter.native="dataFormSubmit()" label-width="100px" size="small" class="body-content">
      <el-form-item label="分组名称" placeholder="分组名称" prop="groupName" class="input-width">
        <el-input v-model="dataForm.groupName" maxlength="30" show-word-limit></el-input>
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
        groupName: ''
      },
      dataRule: {
        groupName: [
          { required: true, message: '请输入分组名称', trigger: 'blur' },
        ]
      }
    }
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0
      this.visible = true
      this.isSubmit = false
      this.dataForm.groupName = null
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl('/admin/user/group/list'),
            method: 'get',
            params: this.$http.adornParams(
              Object.assign({ id: this.dataForm.id, current: 1 }), false
            )
          }).then(({ data }) => {
            this.dataForm.groupName = data.records[0].groupName
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
              url: this.$http.adornUrl('/admin/user/group/update'),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id,
                'groupName': this.dataForm.groupName
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
            console.log('get')
            this.$http({
              url: this.$http.adornUrl('/admin/user/group/add'),
              method: 'get',
              params: this.$http.adornParams({
                id: this.dataForm.id,
                groupName: this.dataForm.groupName
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

.input-width {
  width: 400px;
}

el-form-item {
  margin-bottom: 80px;
}
</style>
