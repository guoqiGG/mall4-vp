<template>
  <div class="lang-国际化配置-set gray-box top-redius border-bottom-gray">
    <div class="title">{{$t('sysManagement.internationalConfig')}}</div>
    <el-form
      ref="dataForm"
      label-width="180px"
      size="mini"
      class="set-form"
      label-position="left"
    >
      <div class="config-til">
        <el-row >
          <!-- 语言名称 -->
          <el-col :span="4">{{ $t('sysManagement.languageName') }}</el-col>
          <!-- 语言代码 -->
          <el-col :span="4" :offset="1">{{ $t('sysManagement.languageCode') }}</el-col>
          <!-- 语言编号 -->
          <el-col :span="4" :offset="1">{{ $t('sysManagement.languageNumber') }}</el-col>
          <!-- 默认语言 -->
          <el-col :span="2" :offset="1">{{ $t('sysManagement.defaultLanguage') }}</el-col>
          <!-- 是否隐藏 -->
          <el-col :span="2" :offset="1">{{ $t('sysManagement.whetherHide') }}</el-col>
          <el-col :span="3" :offset="1">{{ $t('remindPop.operation') }}</el-col>
        </el-row>
      </div>
      <template v-for="(item,index) in langItemList">

        <el-row>
          <!-- 语言名称 -->
          <el-col :span="4">
            <el-form-item label="" label-width="0">
              <el-input v-model.trim="item.name" :placeholder="$t('sysManagement.languageName')" maxlength="30" ></el-input>
              <div v-show="submitStatus&&!item.name" class="el-form-item__error tips-con">{{ $t('sysManagement.languageNameNotNull') }}</div>
            </el-form-item>
          </el-col>
          <!-- 语言代码 -->
          <el-col :span="4" :offset="1">
            <el-form-item label="" label-width="0">
              <el-input v-model.trim="item.language" :placeholder="$t('sysManagement.languageCode')" @change="languageChange(item)" maxlength="20"></el-input>
              <div v-show="submitStatus&&!item.language" class="el-form-item__error tips-con">{{ $t('sysManagement.languageCodeNotNull') }}</div>
              <div v-show="item.language&&!!item.isFormalError" class="el-form-item__error tips-con">{{ $t('sysManagement.formatErrorNotContainChinese') }}</div>
            </el-form-item>
          </el-col>
          <!-- 语言编号 -->
          <el-col :span="4" :offset="1">
            <el-form-item label="" label-width="0">
              <el-input v-model.trim="item.lang" type="number" :min="0" :placeholder="$t('sysManagement.languageNumber')" @change="langChange(index)"></el-input>
              <div v-show="submitStatus&&item.lang===''" class="el-form-item__error tips-con">{{ $t('sysManagement.languageNumberNotNull') }}</div>
            </el-form-item>
          </el-col>
          <!-- 默认语言 -->
          <el-col :span="2" :offset="1">
            <el-form-item label="" label-width="0">
              <el-switch v-model="item.master" active-color="#13ce66" inactive-color="#ff4949" :active-value="1" :inactive-value="0"
              @change="(c)=>masterChange(c,index)" :disabled="item.master === 1"></el-switch>
            </el-form-item>
          </el-col>
          <!-- 是否隐藏 -->
          <el-col :span="2" :offset="1">
            <el-form-item label="" label-width="0">
              <el-switch v-model="item.hide" active-color="#13ce66" inactive-color="#ff4949" :active-value="1" :inactive-value="0"
              @change="(c)=>hideChange(c,item)"  :disabled="masterLangInfo.lang === item.lang" ></el-switch>
            </el-form-item>
          </el-col>
          <el-col :span="3" :offset="1">
            <el-form-item label="" label-width="0">
              <span class="del-btn" @click="delLang(item,index)">{{$t('user.deleteBtn')}}</span>
            </el-form-item>
          </el-col>
        </el-row>
      </template>
      <div class="default-btn primary-btn" @click="addLang">{{$t('sysManagement.add')}}</div>
      <div class="default-btn primary-btn" @click="dataFormSubmit()">{{$t('sysManagement.save')}}</div>
    </el-form>
  </div>
</template>

<script>
export default {
  data () {
    return {
      masterLangInfo: {
        name: '',
        lang: '',
        language: ''
      },
      // lang:语言编号 master:主语言
      langItemList: [
        {name: '中文', lang: 0, master: 1, language: 'zh', hide: 0}
      ],
      submitStatus: false
    }
  },
  mounted () {
    this.init()
  },
  methods: {
    init () {
      this.$nextTick(() => {
        this.$http({
          url: this.$http.adornUrl('/sys/pconfig/info/' + 'INTERNATIONALIZATION_CONFIG'),
          method: 'get'
        }).then(({ data }) => {
          console.log('初始化国际化配置', data)
          if (data) {
            const info = this.dataForm = JSON.parse(data)
            this.$store.commit('lang/updateLang', info)
            this.masterLangInfo.name = info.name
            this.masterLangInfo.lang = info.lang
            this.masterLangInfo.language = info.language
            this.langItemList = info.langItemList.map(m => {
              const reg = /[^A-Za-z_-]/g
              m.isFormalError = reg.test(m.language)
              return m
            })
          }
        })
      })
    },
    // 限制语言代码输入
    languageChange (item) {
      const reg = /[^A-Za-z_-]/g
      this.$set(item, 'isFormalError', reg.test(item.language))
    },
    // 语言编码限制
    langChange (inx) {
      let val = this.langItemList[inx].lang || 0
      val = Math.trunc(Math.abs(val))
      if (val > 1000) {
        val = 1000
      }
      this.langItemList[inx].lang = val
    },
    // 添加语言
    addLang () {
      this.langItemList.push({name: '', lang: '', master: 0, language: '', hide: 0})
    },
    // 删除语言
    delLang (item, index) {
      if (item.master) {
        this.$message({
          message: this.$i18n.t('sysManagement.mainLanguageNotDelete'),
          type: 'error',
          duration: 1000
        })
        return
      }
      this.langItemList.splice(index, 1)
    },
    // 设置主语言
    masterChange (c, inx) {
      // 主语言不可隐藏
      this.langItemList[inx].hide = 0
      // 主语言信息
      this.masterLangInfo.name = this.langItemList[inx].name
      this.masterLangInfo.lang = this.langItemList[inx].lang
      this.masterLangInfo.language = this.langItemList[inx].language
      this.langItemList.forEach((item, index) => {
        if (index !== inx && item.master === 1) {
          item.master = 0
          item.hide = 0
        }
      })
    },
    // 隐藏限制
    hideChange (c, curItem) {
      if (c === 1 && curItem.master === 1) {
        curItem.hide = 0
      }
    },
    // 验证是否可提交
    isSubmit () {
      let langArr = []
      for (const item of this.langItemList) {
        if (item.name === '' || item.lang === '' || item.language === '') {
          return false
        }
        langArr.push(item.lang)
      }
      const reLangArr = [...new Set(langArr)]
      if (reLangArr.length !== langArr.length) {
        this.$message({
          message: this.$i18n.t('sysManagement.duplicateLanguage'),
          type: 'error',
          duration: 1500
        })
        return false
      }
      for (const item of this.langItemList) {
        if (item.isFormalError) {
          return false
        }
      }
      return true
    },
    // 表单提交
    dataFormSubmit () {
      console.log(this.langItemList)
      this.submitStatus = true
      if (!this.isSubmit()) {
        return
      }
      for (const langItem of this.langItemList) {
        if (langItem.master) {
          this.masterLangInfo = {
            name: langItem.name,
            lang: langItem.lang,
            language: langItem.language
          }
          break
        }
      }
      const paramsData = {
        ...this.masterLangInfo,
        langItemList: this.langItemList
      }
      console.log(paramsData)
      this.$http({
        url: this.$http.adornUrl('/sys/pconfig/save'),
        method: 'post',
        data: this.$http.adornData({
          'paramKey': 'INTERNATIONALIZATION_CONFIG',
          'paramValue': JSON.stringify(paramsData),
          'remark': this.$i18n.t('sysManagement.internationalConfig')
        })
      }).then(({ data }) => {
        console.log('提交结果', data)
        this.init()
        this.$message({
          message: this.$i18n.t('publics.operation'),
          type: 'success',
          duration: 1500
        })
      })
    }
  }
}
</script>
<style scoped>
.config-til{
  margin-bottom: 12px;
}
.del-btn{
  color: #F81A1A;
  cursor: pointer;
}
.add-btn{
  background-color: #409eff;
  color: #fff;
}
.tips-con{
  width: 100%;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
</style>
