<template>
  <!-- 发货信息，用于导出代发货订单的excel交给快递公司 -->
  <el-dialog :modal="false" title="请选择操作,同时只能上传一个文件" :close-on-click-modal="false" :visible.sync="visible"
    width="38%" class="box">
    <el-upload ref="upload" v-loading="loading" :auto-upload="false" :v-loading="loading"
      :before-upload="beforeAvatarUpload" :file-list="files" :limit="1" :on-error="uploadFalse"
      :on-preview="handlePreview" :on-remove="handleRemove" :on-success="uploadSuccess"
      :action="this.$http.adornUrl('/platform/order/ship/goods/exportOrderExcel')" class="upload-demo"
      name="orderExcelFile">
      <!-- :headers="{ Authorization: $cookie.get('bbcAuthorization_vs'), locale: lang }" -->
      <!-- :file-list="fileList" -->
      <!-- multiple -->
      <div slot="tip" class="el-upload__tip" />
      <div style="margin-left: 20px" class="default-btn" @click="submitUpload">{{ $t("order.ImportingFiles") }}</div>
      <div slot="trigger" class="default-btn primary-btn">{{
        $t("order.SelectFile")
      }}</div>
    </el-upload>
  </el-dialog>
</template>
<script>
// import * as api from '@/api/product/list'
// import { getToken } from '@/utils/auth'
// import * as orderApi from '@/api/order/order'
export default {
  props: {
    param: {
      type: Object,
      default: () => {
        return {}
      }
    }
  },
  data() {
    return {
      lang: localStorage.getItem('bbcLang') || 'zh_CN',
      visible: false,
      loading: false,
      upload: false,
      resourcesUrl: process.env.VUE_APP_RESOURCES_URL,
      couponId: 3,
      files: [],
      getToken: null,
      dataForm: {
        consignmentName: '',
        consignmentMobile: '',
        consignmentAddr: ''
      },
      dataRule: {
        consignmentName: [
          { required: true, message: this.$i18n.t('publics.noNull'), trigger: 'blur' }
        ],
        consignmentMobile: [
          { required: true, message: this.$i18n.t('publics.noNull'), trigger: 'blur' }
        ],
        consignmentAddr: [
          { required: true, message: this.$i18n.t('publics.noNull'), trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    uploadSuccess(response) {
      // console.log(response)
      // const data = response
      // if (!data) {
      //   alert(this.$i18n.t('order.fileSuccess'))
      // } else {
      //   alert(data)
      // }
      this.loading = false
      alert(response.msg || '上传成功')
      this.files = []
      this.visible = false
      this.$emit('refreshDataList1')
    },
    uploadFalse(response) {
      this.loading = false
      alert('文件上传失败')
    },
    init(id) {
      this.visible = true
      this.loading = false
      this.files = []
      this.$nextTick(() => {
        // this.$refs['dataForm'].resetFields()
      })
      this.couponId = id
    },
    // 上传前对文件的大小的判断
    beforeAvatarUpload(file) {
      this.upload = true
      const extension = file.name.split('.')[1] === 'xls'
      const extension2 = file.name.split('.')[1] === 'xlsx'
      const isLt2M = file.size / 1024 / 1024 < 10
      if (!extension && !extension2) {
        alert(this.$i18n.t('order.downloadTemplateTips1'))
        return false
      }
      if (!isLt2M) {
        alert(this.$i18n.t('order.downloadTemplateTips2'))
        return false
      }
      this.loading = true
      return extension || (extension2 && isLt2M)
    },
    submitUpload() {
      this.upload = false
      this.$refs.upload.submit()
      if (!this.upload) {
        this.$message.error('请选择要上传的文件')
      }
    },
    handleRemove(file) {
    },
    handlePreview(file) {
      if (file.response.status) {
        alert('文件导入成功')
      } else {
        alert('文件导入失败')
      }
      this.visible = false
      this.$emit('refreshDataList1')
    }
  }
}
</script>
<style lang="scss" scoped>
::v-deep .el-dialog__body {
  padding: 10px 20px;
  padding-bottom: 30px;
}

.tips {
  margin-bottom: 30px;
}
</style>
