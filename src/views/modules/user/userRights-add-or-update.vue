<template>
  <el-dialog
    :title="!dataForm.rightsId ? $t('brand.add') : $t('brand.revise')"
    :close-on-click-modal="false"
    :visible.sync="visible"
    width="550px"
    class='user-right-container'
  >
    <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm" label-width="auto" size="small">
      <el-form-item :label="$t('userRights.rightsNameCn')+':'" prop="rightsNameCn">
        <el-input
          v-model="dataForm.rightsNameCn"
          :placeholder="$t('userRights.enterContentCn')"
          maxlength="5"
          show-word-limit
        ></el-input>
      </el-form-item>
      <el-form-item :label="$t('userRights.rightsNameEn')+':'" prop="rightsNameEn">
        <el-input
          v-model="dataForm.rightsNameEn"
          :placeholder="$t('userRights.enterContentEn')"
          maxlength="20"
          show-word-limit
        ></el-input>
      </el-form-item>
      <el-form-item :label="$t('userRights.rightsIntroduceCn')+':'" prop="descriptionCn">
        <el-input
          v-model="dataForm.descriptionCn"
          type="textarea"
          :autosize="{ minRows: 2}"
          maxlength="250"
          show-word-limit
        ></el-input>
      </el-form-item>
      <el-form-item :label="$t('userRights.rightsIntroduceEn')+':'" prop="descriptionEn">
        <el-input
          v-model="dataForm.descriptionEn"
          type="textarea"
          :autosize="{ minRows: 2}"
          maxlength="250"
          show-word-limit
        ></el-input>
      </el-form-item>
      <el-form-item :label="$t('userRights.rightsIcon')+':'" prop="icon">
        <img-upload v-model="dataForm.icon"></img-upload>
      </el-form-item>
      <el-form-item :label="$t('brand.sequence')+':'" prop="seq">
        <el-input
          v-model="dataForm.seq"
          :placeholder="$t('distribution.errorIntegerTip1')"
          type="number"
          precision="0"
          @change="setShopGetScore"
        ></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <div class="default-btn" @click="visible = false">{{$t('remindPop.cancel')}}</div>
      <div class="default-btn primary-btn" @click="dataFormSubmit()">{{$t('remindPop.confirm')}}</div>
    </span>
  </el-dialog>
</template>
<script>
import ImgUpload from '@/components/img-upload'
export default {
  components: {
    ImgUpload
  },
  data () {
    var validateName = (rule, value, callback) => {
      if (!value || !value.trim()) {
        callback(new Error(this.$i18n.t('userRights.nameCanNotEmptyCn')))
      } else {
        callback()
      }
    }
    var validateDescription = (rule, value, callback) => {
      if (!value || !value.trim()) {
        callback(new Error(this.$i18n.t('userRights.intrCanNotEmpty')))
      } else {
        callback()
      }
    }
    return {
      visible: false,
      dataForm: {
        rightsId: null,
        rightsNameCn: null,
        rightsNameEn: null,
        icon: null,
        description: null,
        descriptionCn: null,
        descriptionEn: null,
        seq: 1
      },
      isSubmit: false,
      dataRule: {
        rightsNameCn: [
          { required: true, message: this.$i18n.t('userRights.nameCanNotEmptyCn'), trigger: 'blur' },
          { validator: validateName, trigger: 'blur' }
        ],
        icon: [
          { required: true, message: this.$i18n.t('userRights.iconCanNotEmpty'), trigger: 'blur' }
        ],
        descriptionCn: [
          { required: true, message: this.$i18n.t('userRights.intrCanNotEmptyCn'), trigger: 'blur' },
          { validator: validateDescription, trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    init (rightsId) {
      this.dataForm.rightsId = rightsId || 0
      this.visible = true
      this.isSubmit = false
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.rightsId) {
          this.$http({
            url: this.$http.adornUrl('/user/userRights/info/' + this.dataForm.rightsId),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({ data }) => {
             if (data?.userRightsLangList) {
                data.userRightsLangList.map(val=>{
                  const rightsName= ['rightsNameCn','rightsNameEn'][val.lang]
                  const description = ['descriptionCn','descriptionEn'][val.lang]
                  data[rightsName]=val.rightsName
                  data[description]=val.description
                })
              }
            this.dataForm = data
          })
        } else {
          // 获取最大的序号
          this.$http({
            url: this.$http.adornUrl('/user/userRights/getMaxSeq'),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({ data }) => {
            this.dataForm.seq = data
          })
        }
      })
    },

    // 顺序数值判断
    setShopGetScore () {
      let num = Math.round(this.dataForm.seq)
      this.dataForm.seq = num < 1 ? 1 : num
      if (num >= 100000000) {
        this.dataForm.seq = 100000000
      }
    },
    /**
      * 表单提交
      */
    dataFormSubmit () {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.isSubmit) {
            return false
          }
          this.isSubmit = true
          this.getVipInternationalization()

          this.$http({
            url: this.$http.adornUrl('/user/userRights'),
            method: this.dataForm.rightsId ? 'put' : 'post',
            data: this.$http.adornData(this.dataForm)
          }).then(({ data }) => {
            this.$message({
              message: this.$i18n.t('remindPop.succeeded'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.visible = false
                this.$emit('refreshDataList')
              }
            })
          }).catch((e) => {
            this.isSubmit = false
          })
        }
      })
    },
    //vip 国际化
    getVipInternationalization(){
      if(this.dataForm?.userRightsLangList){
        this.dataForm.userRightsLangList.map((val)=>{
          if(val.lang===0){
            val.rightsName= this.dataForm.rightsNameCn
            val.description= this.dataForm.descriptionCn
          }
          if(val.lang===1){
            val.rightsName= this.dataForm.rightsNameEn
            val.description= this.dataForm.descriptionEn
          }
        })
      }else{
        this.dataForm.userRightsLangList = [
        {
          lang: 0,
          rightsName: this.dataForm.rightsNameCn,
          description: this.dataForm.descriptionCn

        },
        {
          lang: 1,
          rightsName: this.dataForm.rightsNameEn,
          description: this.dataForm.descriptionEn
        }
      ]}
    }
  }
}
</script>

<style lang="scss" scoped>
.user-right-container{
  ::v-deep .el-textarea__inner{
    padding-right:15%
  }
}
</style>
