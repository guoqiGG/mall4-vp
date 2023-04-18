<template>
  <el-dialog
    :title="
      !dataForm.categoryId
        ? this.$i18n.t('brand.add')
        : this.$i18n.t('brand.revise')
    "
    :close-on-click-modal="false"
    :visible.sync="visible"
    @close="close"
    width="600px"
  >
    <el-form @submit.native.prevent
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="180px"
      size="small"
      class="form-box"
    >
      <!-- :inline="true" -->
      <el-form-item
        v-if="dataForm.type !== 2 && isSecondLevel === false"
        :label="this.$i18n.t('category.categoryPicture')"
        prop="pic"
        :required="isRequiredImg"
      >
        <img-upload v-model="dataForm.pic"></img-upload>
        <span v-if="dataForm.parentId === 0"
          >{{ $t("category.recPicSize") }}532*160</span
        >
        <span v-if="dataForm.parentId !== 0"
          >{{ $t("category.recPicSize") }}120*120</span
        >
      </el-form-item>
      <!-- 语言选择 -->
      <el-form-item :label="$t('product.selectLanguage')" v-if="langItemList.length > 1">
        <el-select v-model="curLang" multiple :placeholder="$t('tip.select')" @change='selectLang' style="width: 100%;">
          <el-option
            v-for="item in langItemList"
            :key="item.lang"
            :label="item.name"
            :value="item.lang">
          </el-option>
        </el-select>
      </el-form-item>
      <!-- 分类名称 -->
      <template v-if="dataForm.type !== 2">
        <div v-for="(item,index) in categoryLangList" :key="index">
          <el-form-item
            :label="$t('publics.category') + (langItemList.length === 1 ? '' : `(${item.langName})`)"
          >
            <el-input
              size="small"
              v-model.trim="item.categoryName"
              maxlength="20"
              show-word-limit
            ></el-input>
          </el-form-item>
        </div>
      </template>
      <!-- 选择上级分类 -->
      <el-form-item :label="this.$i18n.t('category.categoryParent')">
        <el-cascader
          size="small"
          expand-trigger="hover"
          :key="cascaderKey"
          :options="categoryList"
          :props="categoryTreeProps"
          :clearable="true"
          v-model="selectedCategory"
          @change="handleChange"
          style="width: 100%"
        ></el-cascader>
      </el-form-item>
      <el-form-item
        v-if="dataForm.type !== 2"
        :label="this.$i18n.t('hotSearch.seq')"
        prop="seq"
      >
        <el-input-number
          class="input-num"
          v-model="dataForm.seq"
          controls-position="right"
          :min="0"
          :max="100000000"
          :label="this.$i18n.t('hotSearch.seq')"
        ></el-input-number>
      </el-form-item>
      <el-form-item v-if="this.selectedCategory.length === 2" :label="this.$i18n.t('category.deductionRate')" prop="deductionRate">
        <el-input-number
          class="input-num"
          v-model.number="dataForm.deductionRate"
          type="number"
          controls-position="right"
          :precision="4"
          :step="0.1"
          :min="0"
          :max="99.9999"
          placeholder="0~99.9999"
        />
        %
      </el-form-item>
      <el-form-item
        :label="this.$i18n.t('product.status')"
        prop="status"
      >
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">{{ $t("brand.offline") }}</el-radio>
          <el-radio :label="1">{{ $t("brand.normal") }}</el-radio>
        </el-radio-group>
      </el-form-item>
    </el-form>
    <div
      slot="footer"
      class="dialog-footer"
    >
      <div class="default-btn" @click="visible = false">{{
        $t("remindPop.cancel")
      }}</div>
      <div class="default-btn primary-btn" @click="dataFormSubmit()">{{
        $t("remindPop.confirm")
      }}</div>
    </div>
  </el-dialog>
</template>
<script>
import { treeDataTranslate, idList } from '@/utils'
import ImgUpload from '@/components/img-upload'
export default {
  data () {
    // 一级分类图片不必传
    const validNoEmptyImg = (rule, value, callback) => {
      if (this.selectedCategory.length > 0 && !value) {
        callback(new Error(this.$i18n.t('category.categoryPicNull')))
      } else {
        callback()
      }
    }
    return {
      visible: false,
      isSecondLevel: false,
      dataForm: {
        categoryId: 0,
        grade: 0,
        categoryName: '',
        categoryNameEn: '',
        seq: 1,
        status: 1,
        parentId: 0,
        pic: '',
        deductionRate: ''
      },
      isRequiredImg: false, // 图片分类一级不必填
      cascaderKey: 'cascaderKey',
      isSubmit: false,
      dataRule: {
        // pic: [
        //   {
        //     required: true,
        //     message: this.$i18n.t('category.categoryPicNull'),
        //     trigger: 'blur'
        //   }
        // ]
        pic: [
          {validator: validNoEmptyImg, trigger: 'blur'}
        ]
      },
      categoryList: [],
      selectedCategory: [],
      categoryTreeProps: {
        value: 'categoryId',
        label: 'categoryName',
        checkStrictly: true
      },
      // 语言列表
      langItemList: [],
      masterLangInfo: {name: '', lang: ''},
      // 当前所选语言
      curLang: [],
      categoryLangList: []
    }
  },
  watch: {
    dataForm: {
      handler () {
        if (this.dataForm.type !== 2 && this.isSecondLevel) {
          this.dataForm.pic = ''
        }
      },
      deep: true // true 深度监听
    },
    selectedCategory (list) {
      this.isRequiredImg = !!list.length
    }
  },
  components: {
    ImgUpload
  },
  methods: {
    init (id) {
      this.isSecondLevel = false
      this.dataForm.parentId = 0
      this.dataForm.categoryId = id || 0
      this.isSubmit = false
      this.dataForm.categoryName = ''
      this.dataForm.categoryNameEn = ''
      this.selectedCategory = []
      this.$http({
        url: this.$http.adornUrl(
          '/prod/category/upAndCurrCategoryList/' + this.dataForm.categoryId
        ),
        method: 'get',
        params: this.$http.adornParams({
          maxGrade: 1,
          status: 1
        })
      })
        .then(({ data }) => {
          this.categoryList = treeDataTranslate(data, 'categoryId', 'parentId')
        })
        .then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
            this.selectedCategory = []
          })
        })
        .then(() => {
          if (this.dataForm.categoryId) {
            // 修改
            this.$http({
              url: this.$http.adornUrl(
                `/prod/category/info/${this.dataForm.categoryId}`
              ),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({ data }) => {
              console.log('修改分类data：', data)
              this.isSecondLevel = data.grade === 1
              this.dataForm.categoryId = data.categoryId
              this.dataForm.categoryName = data.categoryName
              this.dataForm.seq = data.seq
              this.dataForm.pic = data.pic
              this.dataForm.parentId = data.parentId
              this.dataForm.status = data.status
              this.dataForm.deductionRate = data.deductionRate
              this.selectedCategory = idList(
                this.categoryList,
                data.parentId,
                'categoryId',
                'children'
              ).reverse()
              this.categoryLangList = data.categoryLangList
              this.getLangList()
            })
          } else {
            this.getLangList()
            this.categoryLangList = []
          }
        })
    },
    // 获取国际化语言列表
    getLangList () {
      this.$http({
        url: this.$http.adornUrl('/sys/lang'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({ data }) => {
        console.log('初始化国际化配置', data)
        if (data) {
          const info = data
          // 主语言信息
          this.masterLangInfo.name = info.name
          this.masterLangInfo.lang = info.lang
          this.langItemList = info.langItemList

          if (this.dataForm.categoryId) {
            // 设置当前选择的语言
            let curLang = []
            let categoryLangList = []
            for (const item of this.langItemList) {
              const fd = this.categoryLangList.find(it => it.lang === item.lang)
              if (fd) {
                fd.langName = item.name
                categoryLangList.push(fd)
                curLang.push(item.lang)
              }
            }
            this.categoryLangList = categoryLangList
            this.selectLang(curLang)
          } else {
            // 设置默认主语言
            this.selectLang([info.lang])
          }
        }
      })
    },
    // 多选框默认选择主语言
    selectLang (value) {
      console.log('当前值', value)
      this.curLang = value
      // 设置主语言不可删除
      if (!this.curLang.includes(this.masterLangInfo.lang)) {
        this.curLang.unshift(this.masterLangInfo.lang)
      }
      // 分类名称
      const tempArr = this.categoryLangList.filter(item => this.curLang.includes(item.lang))
      this.curLang.forEach((item, index) => {
        if (!tempArr.find(f => f.lang == item)) {
          const fd = this.langItemList.find(it => it.lang === item)
          if (fd) {
            tempArr.splice(index, 0, {langName: fd.name, lang: item, categoryId: this.dataForm.categoryId, categoryName: ''})
          }
        }
      })
      this.categoryLangList = tempArr
    },
    handleChange (val) {
      this.isSecondLevel = val.length === 1
      this.dataForm.parentId = val[val.length - 1]
      if (!this.dataForm.parentId && this.dataForm.parentId !== 0) {
        this.dataForm.parentId = 0
      }
      this.cascaderKey = Math.random() * 10000000000000
    },
    // 表单提交
    dataFormSubmit () {
      console.log(this.selectedCategory.length)
      if (this.selectedCategory.length === 0) {
        this.dataForm.grade = 0
      }
      if (this.selectedCategory.length === 1) {
        this.dataForm.grade = 1
      }
      if (this.selectedCategory.length === 2) {
        this.dataForm.grade = 2
      }
      for (const item of this.categoryLangList) {
        if (!item.categoryName) {
          this.$message.error(this.$i18n.t('publics.categoryNoNull'))
          return
        }
      }
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.isSubmit) {
            return false
          }
          this.isSubmit = true
          this.$http({
            url: this.$http.adornUrl(`/prod/category`),
            method: this.dataForm.categoryId ? 'put' : 'post',
            data: this.$http.adornData({
              categoryId: this.dataForm.categoryId || undefined,
              status: this.dataForm.status,
              seq: this.dataForm.seq,
              grade: this.dataForm.grade,
              parentId: this.dataForm.parentId,
              deductionRate: this.dataForm.deductionRate,
              pic: this.dataForm.pic,
              categoryLangList: this.categoryLangList
            })
          })
            .then(({ data }) => {
              this.$message({
                message: this.$i18n.t('remindPop.success'),
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.visible = false
                  this.$emit('refreshDataList')
                  setTimeout(() => {
                    this.dataForm.pic = ''
                  }, 500)
                }
              })
            })
            .catch((e) => {
              this.isSubmit = false
            })
        }
      })
    },
    close () {
      this.dataForm = {
        categoryId: 0,
        grade: 0,
        categoryName: '',
        categoryNameEn: '',
        seq: 1,
        status: 1,
        parentId: 0,
        pic: '',
        deductionRate: ''
      }
    }
  }
}
</script>
<style lang="scss">
.input-num .el-input__inner{
  padding-right: 40px !important;
}
</style>
<style scoped>
.form-box /deep/ .el-form-item__label{
  text-overflow: ellipsis;
  -o-text-overflow: ellipsis;
  -webkit-text-overflow: ellipsis;
  -moz-text-overflow: ellipsis;
  word-break: break-word;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  line-height: 20px;
  padding-top: 6px;
}
</style>
