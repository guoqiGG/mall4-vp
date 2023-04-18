<template>
  <!-- <el-dialog
    :title="!dataForm.id ? this.$i18n.t('sysManagement.add') : this.$i18n.t('sysManagement.modify')"
    :close-on-click-modal="false"
    top="10vh"
    :visible.sync="visible"
    class="el-dialog-body"
    > -->
    <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="160px">
      <!--后台相关配置-->
      <span v-if="paramKey=='PLATFROM_WEBSITE_CONFIG' || paramKey=='MULTISHOP_WEBSITE_CONFIG'">

        <el-form-item :label="this.$i18n.t('webInfoConfig.loginLogo')" prop="bsLoginLogoImg">
          <imgs-upload v-model="dataForm.bsLoginLogoImg" :limit="1" :size="'380*96'" />
          <div class="el-form-item-tips">{{ $t("webInfoConfig.imgsTip") + '380*96' + $t("webInfoConfig.px") }}</div>
        </el-form-item>

        <el-form-item :label="this.$i18n.t('webInfoConfig.loginBg')" prop="bsLoginBgImg">
          <imgs-upload v-model="dataForm.bsLoginBgImg" :limit="1" :size="'1920*940'" />
          <div class="el-form-item-tips">{{ $t("webInfoConfig.imgsTip") + '1920*940' + $t("webInfoConfig.px") }}</div>
        </el-form-item>

        <el-form-item :label="this.$i18n.t('webInfoConfig.pcTitleImg')" prop="bsTitleImg">
          <imgs-upload v-model="dataForm.bsTitleImg" :limit="1" :size="'100*100'"  />
          <div class="el-form-item-tips">{{ $t("webInfoConfig.imgsTip") + '100*100' + $t("webInfoConfig.px") }}</div>
        </el-form-item>

        <el-form-item :label="this.$i18n.t('webInfoConfig.TopIcon')" prop="bsTopBarIcon">
          <imgs-upload v-model="dataForm.bsTopBarIcon" :limit="1" :size="'160*48'"  />
          <div class="el-form-item-tips">{{ $t("webInfoConfig.imgsTip") + '160*48' + $t("webInfoConfig.px") }}</div>
        </el-form-item>

        <el-form-item :label="this.$i18n.t('product.selectLanguage')" prop="selectLanguage" v-if="langItemList && langItemList.length > 1">
          <el-select v-model="curLang" multiple :placeholder="$t('tip.select')" @change='selectLang' style="width: 500px;">
            <el-option
              v-for="item in langItemList"
              :key="item.lang"
              :label="item.name"
              :value="item.lang">
            </el-option>
          </el-select>
        </el-form-item>

        <div v-for="item in webConfigLangList">
          <el-form-item :label="$i18n.t('webInfoConfig.bsCopyright') + (langItemList && langItemList.length > 1 ? `(${item.langName})` : '')" prop="bsCopyright">
            <el-input v-model="item.bsCopyright" size="small"></el-input>
          </el-form-item>
        </div>

        <div v-for="item in webConfigLangList">
          <el-form-item :label="$i18n.t('webInfoConfig.bsTitleContent') + (langItemList && langItemList.length > 1 ? `(${item.langName})` : '')" prop="bsTitleContent">
            <el-input v-model="item.bsTitleContent"  maxlength="20" size="small" show-word-limit></el-input>
          </el-form-item>
        </div>

        <div v-for="item in webConfigLangList">
          <el-form-item :label="$i18n.t('webInfoConfig.bsMenuTitleOpen') + (langItemList && langItemList.length > 1 ? `(${item.langName})` : '')" prop="bsMenuTitleOpen">
            <el-input v-model="item.bsMenuTitleOpen" maxlength="20" size="small" show-word-limit></el-input>
          </el-form-item>
        </div>

        <div v-for="item in webConfigLangList">
          <el-form-item :label="$i18n.t('webInfoConfig.bsMenuTitleClose') + (langItemList && langItemList.length > 1 ? `(${item.langName})` : '')" prop="bsMenuTitleClose">
            <el-input v-model="item.bsMenuTitleClose" maxlength="6" size="small" show-word-limit></el-input>
          </el-form-item>
        </div>
      </span>

      <!--PC端相关配置-->
      <span v-if="paramKey=='PC_WEBSITE_CONFIG'">

        <el-form-item :label="this.$i18n.t('webInfoConfig.logo')" prop="pcLogoImg">
          <imgs-upload v-model="dataForm.pcLogoImg" :limit="1" :size="'100*100'" />
          <div class="el-form-item-tips">{{ $t("webInfoConfig.imgsTip") + '100*100' + $t("webInfoConfig.px") }}</div>
        </el-form-item>

        <el-form-item :label="this.$i18n.t('webInfoConfig.loginBg')" prop="pcLoginBgImg">
          <imgs-upload v-model="dataForm.pcLoginBgImg" :limit="1" :size="'1920*600'" />
          <div class="el-form-item-tips">{{ $t("webInfoConfig.imgsTip") + '1920*600' + $t("webInfoConfig.px") }}</div>
        </el-form-item>

        <el-form-item :label="this.$i18n.t('webInfoConfig.pcQrcodeImg')" prop="pcQrcodeImg">
          <imgs-upload v-model="dataForm.pcQrcodeImg" :limit="1" :size="'100*100'" />
          <div class="el-form-item-tips">{{ $t("webInfoConfig.imgsTip") + '100*100' + $t("webInfoConfig.px") }}</div>
        </el-form-item>

        <el-form-item :label="this.$i18n.t('webInfoConfig.pcTitleImg')" prop="pcTitleImg">
          <imgs-upload v-model="dataForm.pcTitleImg" :limit="1" :size="'100*100'" />
          <div class="el-form-item-tips">{{ $t("webInfoConfig.imgsTip") + '100*100' + $t("webInfoConfig.px") }}</div>
        </el-form-item>

        <el-form-item :label="this.$i18n.t('webInfoConfig.pcLogoImgText')" prop="pcLogoImgText">
          <imgs-upload v-model="dataForm.pcLogoImgText" :limit="1" :size="'380*96'" />
          <div class="el-form-item-tips">{{ $t("webInfoConfig.imgsTip") + '380*96' + $t("webInfoConfig.px") }}</div>
        </el-form-item>

        <el-form-item :label="this.$i18n.t('product.selectLanguage')" prop="selectLanguage" v-if="langItemList && langItemList.length > 1">
          <el-select v-model="curLang" multiple :placeholder="$t('tip.select')" @change='selectLang' style="width: 500px;">
            <el-option
              v-for="item in langItemList"
              :key="item.lang"
              :label="item.name"
              :value="item.lang">
            </el-option>
          </el-select>
        </el-form-item>

        <div v-for="item in webConfigLangList">
          <el-form-item
            :label="$i18n.t('webInfoConfig.bsCopyright') + (langItemList && langItemList.length > 1 ? `(${item.langName})` : '')"
          >
            <el-input v-model="item.pcCopyright" size="small"></el-input>
          </el-form-item>
        </div>

        <div v-for="item in webConfigLangList">
          <el-form-item
            :label="$i18n.t('webInfoConfig.pcCompanyName') + (langItemList && langItemList.length > 1 ? `(${item.langName})` : '')"
          >
            <el-input v-model="item.pcCompanyName" size="small"></el-input>
          </el-form-item>
        </div>

        <div v-for="item in webConfigLangList">
          <el-form-item
            :label="$i18n.t('webInfoConfig.pcCompanyInfo') + (langItemList && langItemList.length > 1 ? `(${item.langName})` : '')"
          >
            <el-input type="textarea" v-model="item.pcCompanyInfo" size="small"></el-input>
          </el-form-item>
        </div>

        <div v-for="item in webConfigLangList">
          <el-form-item
            :label="$i18n.t('webInfoConfig.pcTitleContent') + (langItemList && langItemList.length > 1 ? `(${item.langName})` : '')"
          >
            <el-input v-model="item.pcTitleContent" size="small"></el-input>
          </el-form-item>
        </div>

        <div v-for="item in webConfigLangList">
          <el-form-item
            :label="$i18n.t('webInfoConfig.pcWelcome') + (langItemList && langItemList.length > 1 ? `(${item.langName})` : '')"
          >
            <el-input v-model="item.pcWelcome" size="small"></el-input>
          </el-form-item>
        </div>
      </span>

      <!--H5端相关配置-->
      <span v-if="paramKey=='H5_WEBSITE_CONFIG'">
        <el-form-item :label="this.$i18n.t('webInfoConfig.loginLogo')" prop="uniLoginLogoImg">
          <imgs-upload v-model="dataForm.uniLoginLogoImg" :limit="1" :size="'100*100'" />
          <div class="el-form-item-tips">{{ $t("webInfoConfig.imgsTip") + '100*100' + $t("webInfoConfig.px") }}</div>
        </el-form-item>
        <el-form-item :label="this.$i18n.t('product.selectLanguage')" prop="selectLanguage" v-if="langItemList && langItemList.length > 1">
          <el-select v-model="curLang" multiple :placeholder="$t('tip.select')" @change='selectLang' style="width: 500px;">
            <el-option
              v-for="item in langItemList"
              :key="item.lang"
              :label="item.name"
              :value="item.lang">
            </el-option>
          </el-select>
        </el-form-item>
        <div v-for="(item,index) in webConfigLangList" :key="index">
          <el-form-item
            :label="$i18n.t('webInfoConfig.uniTitleContent') + (langItemList && langItemList.length > 1 ? `(${item.langName})` : '')"
          >
            <el-input
              size="small"
              v-model.trim="item.uniTitleContent"
            ></el-input>
          </el-form-item>
        </div>
        <!-- <el-form-item :label="this.$i18n.t('webInfoConfig.uniTitleContent')+'-'+this.$i18n.t('webInfoConfig.chinese')" prop="uniTitleContentCn">
          <el-input v-model="dataForm.uniTitleContentCn" size="small"></el-input>
        </el-form-item>
        <el-form-item :label="this.$i18n.t('webInfoConfig.uniTitleContent')+'-'+this.$i18n.t('webInfoConfig.english')" prop="uniTitleContentEn">
          <el-input v-model="dataForm.uniTitleContentEn" size="small"></el-input>
        </el-form-item> -->
      </span>

      <!--自提点端相关配置-->
      <span v-if="paramKey=='STATION_WEBSITE_CONFIG'">
        <el-form-item :label="this.$i18n.t('webInfoConfig.loginLogo')" prop="stationLoginLogoImg">
          <imgs-upload v-model="dataForm.stationLoginLogoImg" :limit="1" :size="'100*100'" />
          <div class="el-form-item-tips">{{ $t("webInfoConfig.imgsTip") + '100*100' + $t("webInfoConfig.px") }}</div>
        </el-form-item>
      </span>

      <!-- <el-form-item :label="this.$i18n.t('webInfoConfig.activationStatus')" prop="isActivity">
        <el-radio-group v-model="dataForm.isActivity">
          <el-radio :label="0">{{$i18n.t('webInfoConfig.close')}}</el-radio>
          <el-radio :label="1">{{$i18n.t('webInfoConfig.open')}}</el-radio>
        </el-radio-group>
      </el-form-item> -->

       <el-form-item>
        <el-button type="primary" size="small" @click="dataFormSubmit()">{{ $i18n.t('webInfoConfig.save') }}</el-button>
      </el-form-item>
    </el-form>



    <!-- <span slot="footer">
      <div class="default-btn" @click="visible = false">{{$i18n.t('webInfoConfig.cancellation')}}</div>
      <div class="default-btn primary-btn" @click="dataFormSubmit()">{{$i18n.t('webInfoConfig.determine')}}</div>
      <el-button @click="visible = false">{{$i18n.t('webInfoConfig.cancellation')}}</el-button>
      <el-button type="primary" @click="dataFormSubmit()">{{$i18n.t('webInfoConfig.determine')}}</el-button>
    </span> -->
  <!-- </el-dialog> -->
</template>

<script>
  import ImgsUpload from '@/components/img-upload'
  export default {
    props: {
      paramKey: {
        type: String,
        default: 'PLATFROM_WEBSITE_CONFIG'
      },
      remark: {
        type: String
      }
      // configId: {
      //   type: Number,
      //   default: 0
      // }
    },
    data () {
      return {
        visible: false,
        dataForm: {
          id: null,
          isActivity: 0,
          bsLoginLogoImg: null,
          bsLoginBgImg: null,
          bsCopyright: null,
          bsTitleContent: null,
          bsTitleImg: null,
          bsTopBarIcon: null,
          bsMenuTitleOpen: null,
          bsMenuTitleClose: null,
          pcLogoImg: null,
          pcCopyright: null,
          pcQrcodeImg: null,
          pcCompanyName: null,
          pcCompanyInfo: null,
          pcTitleContent: null,
          pcTitleImg: null,
          pcCompanyNameShort: null,
          pcLogoImgText: null,
          pcWelcome: null,
          pcLoginBgImg: null,
          uniLoginLogoImg: null,
          stationLoginLogoImg: null,
          webConfigLangList: []
        },
        dataRule: {},
        // 语言列表
        langItemList: [],
        masterLangInfo: {name: '', lang: ''},
      // 当前所选语言
        curLang: [],
        webConfigLangList: []
      }
    },
    components: {
      ImgsUpload
    },
    mounted () {
      // this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        this.getLangList()
        this.getConfigData()
      })
    },
    methods: {
      getConfigData () {
        if (this.paramKey) {
          this.$http({
            url: this.$http.adornUrl(`/sys/webConfig/info/${this.paramKey}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data) {
              this.dataForm = data
              this.webConfigLangList = this.dataForm.webConfigLangList
              this.getLangList()
            }
          })
        }
      },
      /**
       * 获取商品配置的语言列表
       */
      getLangList () {
        this.$http({
          url: this.$http.adornUrl('/sys/lang'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({ data }) => {
          console.log('初始化国际化配置', data)
          if (data) {
            const info = data
            this.masterLangInfo.name = info.name
            this.masterLangInfo.lang = info.lang
            this.langItemList = info.langItemList

            if (!this.webConfigLangList) {
              this.webConfigLangList = []
            }

            let curLang = []
            let webConfigLangList = []
            for (const item of this.langItemList) {
              const fd = this.webConfigLangList.find(it => it.lang === item.lang)
              if (fd) {
                fd.langName = item.name
                webConfigLangList.push(fd)
                curLang.push(item.lang)
              }
            }
            this.webConfigLangList = webConfigLangList
            this.selectLang(curLang)
          }
        })
      },
      selectLang (value) {
        console.log('当前值', value)
        this.curLang = value
        // 设置主语言不可删除
        if (!this.curLang.includes(this.masterLangInfo.lang)) {
          this.curLang.unshift(this.masterLangInfo.lang)
        }
        // 菜单名称
        const tempArr = this.webConfigLangList.filter(item => this.curLang.includes(item.lang))
        this.curLang.forEach((item, index) => {
          if (!tempArr.find(f => f.lang == item)) {
            const fd = this.langItemList.find(it => it.lang === item)
            if (fd) {
              tempArr.splice(index, 0, {
                lang: fd.lang,
                langName: fd.name,
                bsCopyright: '',
                bsTitleContent: '',
                bsMenuTitleOpen: '',
                bsMenuTitleClose: '',
                pcCopyright: '',
                pcCompanyName: '',
                pcCompanyInfo: '',
                pcTitleContent: '',
                pcCompanyNameShort: '',
                pcWelcome: '',
                uniTitleContent: ''
              })
            }
          }
        })
        this.webConfigLangList = tempArr
      },
      checkLang () {
        for (var i = 0; i < this.webConfigLangList.length; i++) {
          const item = this.webConfigLangList[i]
          // 后台配置判断
          if (this.paramKey === 'PLATFROM_WEBSITE_CONFIG' || this.paramKey === 'MULTISHOP_WEBSITE_CONFIG') {
            if (!item.bsCopyright) {
              this.errorMsg(this.$i18n.t('webInfoConfig.bsCopyright') + (this.langItemList.length === 1 ? ' ' : '(' + item.langName + ') ') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
              return false
            }
            if (!item.bsTitleContent) {
              this.errorMsg(this.$i18n.t('webInfoConfig.bsTitleContent') + (this.langItemList.length === 1 ? ' ' : '(' + item.langName + ') ') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
              return false
            }
            if (!item.bsMenuTitleOpen) {
              this.errorMsg(this.$i18n.t('webInfoConfig.bsMenuTitleOpen') + (this.langItemList.length === 1 ? ' ' : '(' + item.langName + ') ') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
              return false
            }
            if (!item.bsMenuTitleClose) {
              this.errorMsg(this.$i18n.t('webInfoConfig.bsMenuTitleClose') + (this.langItemList.length === 1 ? ' ' : '(' + item.langName + ') ') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
              return false
            }
          }

          // pc端配置判断
          if (this.paramKey === 'PC_WEBSITE_CONFIG') {
            if (!item.pcCopyright) {
              this.errorMsg(this.$i18n.t('webInfoConfig.bsCopyright') + (this.langItemList.length === 1 ? ' ' : '(' + item.langName + ') ') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
              return false
            }
            if (!item.pcCompanyName) {
              this.errorMsg(this.$i18n.t('webInfoConfig.pcCompanyName') + (this.langItemList.length === 1 ? ' ' : '(' + item.langName + ') ') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
              return false
            }
            if (!item.pcCompanyInfo) {
              this.errorMsg(this.$i18n.t('webInfoConfig.pcCompanyInfo') + (this.langItemList.length === 1 ? ' ' : '(' + item.langName + ') ') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
              return false
            }
            if (!item.pcTitleContent) {
              this.errorMsg(this.$i18n.t('webInfoConfig.pcTitleContent') + (this.langItemList.length === 1 ? ' ' : '(' + item.langName + ') ') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
              return false
            }
            if (!item.pcWelcome) {
              this.errorMsg(this.$i18n.t('webInfoConfig.pcWelcome') + (this.langItemList.length === 1 ? ' ' : '(' + item.langName + ') ') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
              return false
            }
          }
          // h5端配置判断
          if (this.paramKey === 'H5_WEBSITE_CONFIG') {
            if (!item.uniTitleContent) {
              this.errorMsg(this.$i18n.t('webInfoConfig.uniTitleContent') + (this.langItemList.length === 1 ? ' ' : '(' + item.langName + ') ') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
              return false
            }
          }
        }
        return true
      },
      checkImage () {
        // 后台配置判断
        if (this.paramKey === 'PLATFROM_WEBSITE_CONFIG' || this.paramKey === 'MULTISHOP_WEBSITE_CONFIG') {
          if (!this.dataForm.bsLoginLogoImg) {
            this.errorMsg(this.$i18n.t('webInfoConfig.loginLogo') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
            return false
          }
          if (!this.dataForm.bsLoginBgImg) {
            this.errorMsg(this.$i18n.t('webInfoConfig.loginBg') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
            return false
          }
          if (!this.dataForm.bsTitleImg) {
            this.errorMsg(this.$i18n.t('webInfoConfig.bsTitleImg') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
            return false
          }
          // TopIcon
          if (!this.dataForm.bsTopBarIcon) {
            this.errorMsg(this.$i18n.t('webInfoConfig.TopIcon') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
            return false
          }
        }

        // pc端配置判断
        if (this.paramKey === 'PC_WEBSITE_CONFIG') {
          console.log('this.dataForm.pcLogoImg', this.dataForm.pcLogoImg)
          if (!this.dataForm.pcLogoImg) {
            this.errorMsg(this.$i18n.t('webInfoConfig.logo') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
            return false
          }
          if (!this.dataForm.pcLoginBgImg) {
            this.errorMsg(this.$i18n.t('webInfoConfig.loginBg') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
            return false
          }
          if (!this.dataForm.pcQrcodeImg) {
            this.errorMsg(this.$i18n.t('webInfoConfig.pcQrcodeImg') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
            return false
          }
          if (!this.dataForm.pcTitleImg) {
            this.errorMsg(this.$i18n.t('webInfoConfig.bsTitleImg') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
            return false
          }
          if (!this.dataForm.pcLogoImgText) {
            this.errorMsg(this.$i18n.t('webInfoConfig.pcLogoImgText') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
            return false
          }
        }
        // h5端配置判断
        if (this.paramKey === 'H5_WEBSITE_CONFIG') {
          if (!this.dataForm.uniLoginLogoImg) {
            this.errorMsg(this.$i18n.t('webInfoConfig.loginLogo') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
            return false
          }
        }

        // 自提点端配置判断
        if (this.paramKey === 'STATION_WEBSITE_CONFIG') {
          if (!this.dataForm.stationLoginLogoImg) {
            this.errorMsg(this.$i18n.t('webInfoConfig.loginLogo') + this.$i18n.t('webInfoConfig.canNotBeEmpty'))
            return false
          }
        }
        return true
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (!valid || !this.checkImage()) {
            return
          }
          if (!this.checkLang() || !this.checkImage()) {
            return
          }
          this.dataForm.webConfigLangList = this.webConfigLangList

          // 新增或修改配置
          const paramsData = {
            id: this.dataForm.id || 0,
            paramKey: this.paramKey,
            paramValue: JSON.stringify(this.dataForm),
            remark: this.remark
          }
          this.$http({
            url: this.$http.adornUrl('/sys/webConfig/save'),
            method: this.dataForm.id ? 'put' : 'post',
            data: this.$http.adornData(paramsData)
          }).then(({data}) => {
            this.$message({
              message: this.$i18n.t('remindPop.success'),
              type: 'success',
              duration: 1500,
              customClass: 'zZindex'
              // onClose: () => {
              //   this.visible = false
              //   this.$emit('refreshDataList')
              // }
            })
            this.updateWebConfigData()
          })
        })
      },

      errorMsg (message) {
        this.$message({
          message: message,
          type: 'error',
          duration: 1500,
          customClass: 'zZindex'
        })
      },

      // 更新网站配置信息
      updateWebConfigData () {
        if (this.paramKey !== 'PLATFROM_WEBSITE_CONFIG') {
          return false
        }
        this.$http({
          url: this.$http.adornUrl('/sys/webConfig/getActivity'),
          method: 'get'
        }).then(({data}) => {
          this.$store.commit('webConfig/addData', data)
        })
      }
    }
  }
</script>

<style scoped>
/* .el-dialog-body .el-dialog__header{
  border-bottom: 1px solid #c0ccda;
}
.el-dialog-body .el-dialog__body {
    height: 650px;
    overflow-y: auto;
}
.el-dialog-body .el-dialog__footer{
  border-top: 1px solid #c0ccda;
}
.zZindex{
  z-index: 9999 !important;
} */

  .el-form-item-tips {
    font-size: 12px;
    color: #999;
    line-height: 1em;
    padding-top: 8px;
    padding-bottom: 5px;
  }
</style>
<style scoped lang="scss">
  ::v-deep .el-input {
    width: 60%;
  }
  ::v-deep .el-textarea {
    width: 60%;
  }
</style>
