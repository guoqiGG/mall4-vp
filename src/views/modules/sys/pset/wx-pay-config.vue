<template>
  <div class="pay-wx-set gray-box top-redius border-bottom-gray">
    <div class="title">{{$t('sysManagement.wechatPaymentConfiguration')}}</div>
    <el-form @submit.native.prevent
      ref="dataForm"
      label-width="180px"
      size="mini"
      class="set-form"
      label-position="left"
      @keyup.enter.native="dataFormSubmit()"
      :rules="dataRule"
      :model="dataForm"
    >
      <el-form-item :label="`${$t('sysManagement.wechatPay')} mchId:`" style="width:640px" prop="mchId">
        <el-input v-model="dataForm.mchId" :placeholder="`${$t('sysManagement.wechatPay')} mchId`" controls-position="right"></el-input>
      </el-form-item>
      <el-form-item :label="`${$t('sysManagement.paymentType')}:`" style="width:640px" prop="version">
        <el-radio-group v-model="dataForm.version">
          <el-radio :label="2">v2</el-radio>
          <el-radio :label="3">v3</el-radio>
        </el-radio-group>
      </el-form-item>
      <!-- v2配置 -->
      <el-form-item :label="`${$t('sysManagement.wechatPay')} mchKey:`" style="width:640px" prop="mchKey" v-if="dataForm.version === 2">
        <el-input v-model="dataForm.mchKey" :placeholder="`${$t('sysManagement.wechatPay')} mchKey`" controls-position="right"></el-input>
      </el-form-item>
      <el-form-item :label="`${$t('sysManagement.paymentCertificatePath')}:`" style="width:640px" prop="keyPath" v-if="dataForm.version === 2">
        <el-input v-model="dataForm.keyPath" :placeholder="`${$t('sysManagement.paymentCertificatePath')}`" controls-position="right"></el-input>
      </el-form-item>
      <!-- v3配置 -->
      <el-form-item :label="`${$t('sysManagement.wechatPay')} apiv3Key:`" style="width:640px" prop="apiv3Key" v-if="dataForm.version === 3">
        <el-input v-model="dataForm.apiv3Key" :placeholder="`${$t('sysManagement.wechatPay')} apiv3Key`" controls-position="right"></el-input>
      </el-form-item>
      <el-form-item :label="`${$t('sysManagement.certificateSerialNumber')}:`" style="width:640px" prop="certSerialNo" v-if="dataForm.version === 3">
        <el-input v-model="dataForm.certSerialNo" :placeholder="`${$t('sysManagement.certificateSerialNumber')}`" controls-position="right"></el-input>
      </el-form-item>
      <el-form-item :label="`${$t('sysManagement.merchant')}apiclient_key.pem:`" style="width:640px" prop="privateKeyPath" v-if="dataForm.version === 3">
        <el-input v-model="dataForm.privateKeyPath " :placeholder="`${$t('sysManagement.merchant')}apiclient_key.pem`" controls-position="right"></el-input>
      </el-form-item>
      <el-form-item :label="`${$t('sysManagement.merchant')}apiclient_cert.pem:`" style="width:640px" prop="privateCertPath" v-if="dataForm.version === 3">
        <el-input v-model="dataForm.privateCertPath" :placeholder="`${$t('sysManagement.merchant')}apiclient_cert.pem`" controls-position="right"></el-input>
      </el-form-item>
      <div class="default-btn" @click="dataFormSubmit()">{{$t('sysManagement.save')}}</div>
    </el-form>
    <!-- <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
    </span>-->
  </div>
</template>

<script>
export default {
  data () {
    return {
      dataForm: {
        // id: null,
        // paramKey: 'WXPAY_CONFIG',
        mchId: null,
        mchKey: null,
        keyPath: null,
        signType: null,
        version: 3,
        apiv3Key: null,
        certSerialNo: null,
        privateKeyPath: null,
        privateCertPath: null
        // paramValue: null
      },
      value: [],
      dataRule: {
        version: [
          { required: true, message: `${this.$i18n.t('sysManagement.paymentType')} ${this.$i18n.t('sysManagement.nullTips')}`, trigger: 'blur' }
        ],
        mchId: [
          { required: true, message: `${this.$i18n.t('sysManagement.wechatPay')} mchId ${this.$i18n.t('sysManagement.nullTips')}`, trigger: 'blur' }
        ],
        mchKey: [
          { required: true, message: `${this.$i18n.t('sysManagement.wechatPay')} mchKey ${this.$i18n.t('sysManagement.nullTips')}`, trigger: 'blur' }
        ],
        keyPath: [
          { required: true, message: `${this.$i18n.t('sysManagement.paymentCertificatePath')} ${this.$i18n.t('sysManagement.nullTips')}`, trigger: 'blur' }
        ],
        apiv3Key: [
          { required: true, message: `${this.$i18n.t('sysManagement.wechatPay')} apiv3Key ${this.$i18n.t('sysManagement.nullTips')}`, trigger: 'blur' }
        ],
        certSerialNo: [
          { required: true, message: `${this.$i18n.t('sysManagement.certificateSerialNumber')} ${this.$i18n.t('sysManagement.nullTips')}`, trigger: 'blur' }
        ],
        privateKeyPath: [
          { required: true, message: `${this.$i18n.t('sysManagement.merchant')}apiclient_key.pem ${this.$i18n.t('sysManagement.nullTips')}`, trigger: 'blur' }
        ],
        privateCertPath: [
          { required: true, message: `${this.$i18n.t('sysManagement.merchant')}apiclient_cert.pem ${this.$i18n.t('sysManagement.nullTips')}`, trigger: 'blur' }
        ]
      }
    }
  },
  mounted () {
    this.init()
  },
  methods: {
    init () {
      // this.$refs['dataForm'].resetFields()
      this.$nextTick(() => {
        this.$http({
          url: this.$http.adornUrl('/sys/pconfig/info/' + 'WXPAY_CONFIG'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({ data }) => {
          if (data) {
            this.dataForm = JSON.parse(data)
          }
        })
        this.$refs['dataForm'].resetFields()
      })
    },
    // 表单提交
    dataFormSubmit () {
      this.$refs['dataForm'].validate((valid) => {
        // this.dataForm.signType = JSON.stringify(this.value)
        if (valid) {
          this.$http({
            url: this.$http.adornUrl('/sys/pconfig/save'),
            method: 'post',
            data: this.$http.adornData({
              'paramKey': 'WXPAY_CONFIG',
              'paramValue': JSON.stringify(this.dataForm),
              'remark': this.$i18n.t('sysManagement.paymentCertificatePath')
            })
          }).then(({ data }) => {
            this.$message({
              message: this.$i18n.t('publics.operation'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.visible = false
                this.$emit('refreshDataList')
              }
            })
          })
        }
      })
    }
  }
}
</script>

