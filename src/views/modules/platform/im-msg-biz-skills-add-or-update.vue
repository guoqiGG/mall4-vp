<template>
  <el-dialog
    :title="!dataForm.id ? $t('brand.add') : $t('user.editBtn')"
    :close-on-click-modal="false"
    :visible.sync="visible"
    @close="dialogClose"
    top="5vh"
    width="780px"
  >
    <el-form @submit.native.prevent
      class="dialog-form"
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      :label-width="this.$i18n.t('language') === 'language' ? '150px' : '80px'"
    >

      <el-form-item :label="$t('weixin.keyword')" prop="keywords">
        <el-input size="small" maxlength="36" v-model="dataForm.keywords"></el-input>
      </el-form-item>
      <el-form-item :label="$t('weixin.replyContent')" prop="content">
        <el-input size="small" maxlength="36" v-model="dataForm.content"></el-input>
      </el-form-item>
      <el-form-item :label="$t('product.remark')" prop="remark">
        <el-input size="small" type="textarea" maxlength="100" v-model="dataForm.remark"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <div class="default-btn" @click="visible = false">{{
        $t("remindPop.cancel")
      }}</div>
      <div class="default-btn primary-btn" @click="dataFormSubmit()">{{
        $t("remindPop.confirm")
      }}</div>
    </span>
  </el-dialog>
</template>

<script>
import TinyMce from '@/components/tiny-mce'
import SelectAccountRange from '@/components/SelectAccountRange'
export default {
  data () {
    var validateKeywords = (rule, value, callback) => {
      if (String(value).trim() === '') {
        callback(new Error(this.$t('weixin.keyword') + this.$t('webInfoConfig.canNotBeEmpty')))
      } else {
        callback()
      }
    }
    var validateContent = (rule, value, callback) => {
      if (String(value).trim() === '') {
        callback(new Error(this.$t('weixin.replyContent') + this.$t('webInfoConfig.canNotBeEmpty')))
      } else {
        callback()
      }
    }
    return {
      visible: false,
      dataForm: {
        id: null,
        keywords: '',
        content: ''

      },
      isSubmit: false,
      dataRule: {
        keywords: [
          { required: true, message: this.$t('weixin.keyword') + this.$t('webInfoConfig.canNotBeEmpty'), trigger: 'blur' },
          {validator: validateKeywords, trigger: 'blur'}
        ],
        content: [
          { required: true, message: this.$t('weixin.replyContent') + this.$t('webInfoConfig.canNotBeEmpty'), trigger: 'blur' },
          {validator: validateContent, trigger: 'blur'}
        ]
      }
    }
  },
  components: {
    TinyMce,
    SelectAccountRange
  },
  methods: {
    init (id = null) {
      this.dataForm.id = id || ''
      this.visible = true
      this.isSubmit = false
      this.dialogClose()
      if (this.dataForm.id) {
        this.$http({
          url: this.$http.adornUrl('/platform/imMsgBizSkills/info/' + id),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({ data }) => {
          this.dataForm = data
        })
      }
    },
    // 表单提交
    dataFormSubmit () {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.isSubmit) {
            return false
          }
          const obj = JSON.parse(JSON.stringify(this.dataForm))
          if (!obj.id) {
            delete obj.id
          }
          this.isSubmit = true
          this.$http({
            url: this.$http.adornUrl('/platform/imMsgBizSkills'),
            method: this.dataForm.id ? 'put' : 'post',
            data: this.$http.adornData(obj)
          }).then(({ data }) => {
            this.$message({
              message: this.$i18n.t('remindPop.success'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.visible = false
                this.$emit('refreshDataList')
                this.dataForm.content = ''
              }
            })
          }).catch((e) => {
            this.isSubmit = false
          })
        }
      })
    },
    // 重置表单
    dialogClose () {
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.dialog-form {
  ::v-deep .el-form-item {
    margin-bottom: 4px;
  }
}
</style>

<style lang="scss" scoped>
.dialog-form {
  ::v-deep .el-form-item {
    margin-bottom: 4px;
  }
  ::v-deep .el-form-item.is-error {
    margin-bottom: 22px;
  }
}
.question-icon {
  position: relative;
  right: 30px;
  // top: -13px;
  display: inline-block;
  width: 14px;
  height: 14px;
  line-height: 14px;
  font-size: 12px;
  text-align: center;
  color: #fff;
  background: #C8C9CC;
  border-radius: 50%;
  margin-left: 5px;
  cursor: default;
}.notice-content {
  ::v-deep .el-form-item__label:before{
    content: '*';
    color: #F56C6C;
    margin-right: 4px;
  }
}
</style>
