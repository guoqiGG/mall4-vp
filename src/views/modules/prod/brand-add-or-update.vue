<template>
  <div class="mod-index-img">
    <el-dialog
      :title="!dataForm.brandId ? this.$i18n.t('brand.add') : this.$i18n.t('brand.revise')"
      :close-on-click-modal="false"
      :visible.sync="visible"
      @close="closeDialogHandle"
      width="570px"
    >
      <el-form :model="dataForm" ref="dataForm" :rules="dataRule" label-width="auto" size="small">
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
        <!-- 品牌名称 -->
        <template v-if="dataForm.type !== 2">
          <div v-for="(item,index) in brandLangList" :key="index">
            <el-form-item
              :label="$t('brand.brandName') + (langItemList.length === 1 ? '' : `(${item.langName})`)"
            >
              <el-input
                size="small"
                v-model.trim="item.name"
                maxlength="20"
                show-word-limit
              ></el-input>
            </el-form-item>
          </div>
        </template>

        <!-- 品牌首字母 -->
        <el-form-item :label="this.$i18n.t('brandPopups.brandInitials')" prop="firstLetter">
          <el-input
            type="text"
            :placeholder="this.$i18n.t('brandPopups.inputBrandInitials')"
            v-model="dataForm.firstLetter"
            maxlength="1"
            show-word-limit
          ></el-input>
        </el-form-item>
        <!-- 品牌logo -->
        <el-form-item :label="this.$i18n.t('brandPopups.brandLogo')" prop="imgUrl">
          <img-upload v-model="dataForm.imgUrl"></img-upload>
        </el-form-item>
        <!-- 平台分类 -->
        <el-form-item :label="this.$i18n.t('product.platformClassification')" prop="categoryIds">
          <el-select v-model="dataForm.categoryIds" multiple class="select-categoryId" :placeholder="$t('tip.select') + $t('product.platformClassification')">
            <el-option v-for="(item,index) in categoryList" :key="index" :label="item.categoryName" :value="item.categoryId"></el-option>
          </el-select>
        </el-form-item>
        <!-- 序号 -->
        <el-form-item :label="this.$i18n.t('brand.sequence')" prop="seq">
          <el-input v-model="dataForm.seq" type="number" :max="32767" @blur="handleSortValue"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <div class="default-btn" @click="visible = false">{{ $t("remindPop.cancel") }}</div>
        <div class="default-btn primary-btn" @click="dataFormSubmit()">{{$t("remindPop.confirm")}}</div>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import ImgUpload from '@/components/img-upload'
import { Debounce } from '@/utils/debounce'
import { validNoEmptySpace } from '@/utils/validate'
export default {
  data () {
    const validateFirstLetter = (rule, value, callback) => {
      const regexp = /^[A-Z]{1}$/
      if (!regexp.test(value)) {
        callback(new Error('请输入一位大写字母'))
      } else {
        callback()
      }
    }
    const validateNotEmpty = (role, value, callback) => {
      if (validNoEmptySpace(value)) {
        callback(new Error(this.$i18n.t('shopProcess.inputAllSpace')))
      } else {
        callback()
      }
    }
    const validateCategory = (role, value, callback) => {
      if (value.length === 0) {
        callback(new Error(this.$i18n.t('user.selectCategory')))
      } else {
        callback()
      }
    }
    return {
      dataForm: {
        brandId: 0,
        imgUrl: '',
        brief: '',
        seq: 0,
        status: 1,
        firstLetter: '',
        categoryIds: []
      },
      brandSelectVisible: false,
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      selectedCategories: [],
      // 分类list
      categoryList: [],
      categorySelectorVisible: false, // 分类弹窗显隐
      dataRule: {
        firstLetter: [
          { required: true, message: this.$i18n.t('brandPopups.emptyBrandInitials'), trigger: 'blur' },
          { validator: validateFirstLetter, trigger: 'blur' }
        ],
        seq: [
          { required: true, message: this.$i18n.t('brandPopups.emptysequence'), trigger: 'blur' }
        ],
        imgUrl: [
          { required: true, message: this.$i18n.t('brandPopups.selectLogo'), trigger: 'blur' }
        ],
        categoryIds: [
          { required: true, message: this.$i18n.t('user.selectCategory'), trigger: 'change', validator: validateCategory }
        ]
      },
      visible: false,
      // 语言列表
      langItemList: [],
      masterLangInfo: {name: '', lang: ''},
      // 当前所选语言
      curLang: [],
      brandLangList: []
    }
  },
  components: {
    ImgUpload
  },
  methods: {
    // 获取分类数据
    init (id) {
      this.selectedCategories = []
      this.visible = true
      this.dataForm.brandId = id || 0
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
      })
      if (this.dataForm.brandId) {
        // 获取产品数据
        this.$http({
          url: this.$http.adornUrl(`/platform/brand/info/${this.dataForm.brandId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({ data }) => {
          this.dataForm = data
          this.dataForm.categoryIds = data.categories ? data.categories.map(x => x.categoryId) : []
          // this.categoryShow(data.categories) // 分类回显
          // 国际化处理
          this.brandLangList = data.brandLangList
          this.getLangList()
          // 获取平台分类
          this.getCategoryList()
        })
      } else {
        // 获取平台分类
        this.getCategoryList()
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          this.dataForm.brandPic = ''
          this.relation = null
        })
        this.getLangList()
        this.brandLangList = []
      }
    },
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
          if (this.dataForm.brandId) {
            let curLang = []
            let brandLangList = []
            for (const item of this.langItemList) {
              const fd = this.brandLangList.find(it => it.lang === item.lang)
              if (fd) {
                fd.langName = item.name
                brandLangList.push(fd)
                curLang.push(item.lang)
              }
            }
            this.brandLangList = brandLangList
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
      const tempArr = this.brandLangList.filter(item => this.curLang.includes(item.lang))
      this.curLang.forEach((item, index) => {
        if (!tempArr.find(f => f.lang == item)) {
          const fd = this.langItemList.find(it => it.lang === item)
          if (fd) {
            tempArr.splice(index, 0, {langName: fd.name, lang: item, brandId: this.dataForm.brandId, name: ''})
          }
        }
      })
      this.brandLangList = tempArr
    },
    // 获取平台分类列表
    getCategoryList () {
      this.$http({
        url: this.$http.adornUrl('/prod/category/listCategoryByGrade'),
        method: 'get',
        params: this.$http.adornParams({
          grade: 0,
          status: 1
        })
      }).then(({ data }) => {
        this.categoryList = data
      })
    },
    /**
     * 分类回显
     */
    // categoryShow (categories) {
    //   if (categories !== null && categories.length) {
    //     var categoryObj = {}
    //     var categoryIds = []
    //     categories.forEach((el, index) => {
    //       categoryIds.push(el.categoryId)
    //       categoryObj.firstCategoryName = el.categories[0].categoryName
    //       categoryObj.secondCategoryName = el.categories[1].categoryName
    //       categoryObj.threeCategoryName = el.categoryName
    //       this.selectedCategories.push(categoryObj)
    //       categoryObj = {}
    //     })
    //     this.dataForm.categoryIds = categoryIds
    //   }
    // },

    // 处理排序输入框
    handleSortValue () {
      if (this.dataForm.seq > 32767) {
        this.$set(this.dataForm, 'seq', 32767)
        return
      }
      if (this.dataForm.seq <= 0 || !this.dataForm.seq) {
        this.$set(this.dataForm, 'seq', 0)
      }
    },

    // 表单提交
    dataFormSubmit: Debounce(function () {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          let param = this.dataForm
          for (const item of this.brandLangList) {
            if (!item.name) {
              this.$message({
                message: this.$i18n.t('brandPopups.emptyBrandName'),
                type: 'error',
                duration: 1500
              })
              return
            }
          }
          param.brandLangList = this.brandLangList
          this.$http({
            url: this.$http.adornUrl(`/platform/brand`),
            method: param.brandId ? 'put' : 'post',
            data: this.$http.adornData(param)
          }).then(({ data }) => {
            this.$message({
              message: this.$i18n.t('remindPop.success'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.visible = false
                this.$emit('refreshDataList', this.page)
              }
            })
          })
        }
      })
    }, 1500),
    // 关闭对话框回调
    closeDialogHandle () {
      this.$refs['dataForm'].resetFields()
      this.dataForm.relation = null
    }
  }
}
</script>
<style lang="scss" scoped>
  .el-input__inner {
    line-height: 1px !important;
  }
  .mod-index-img {
    .select-categoryId {
      width: 60%;
    }
  }
</style>
