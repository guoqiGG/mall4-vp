<template>
  <div class="mod-prod-info">
    <!-- 步骤 -->
    <div class="post-step">
      <div class="step-item" :class="{ 'active': postingSteps === 1 || postingSteps === 2 }">
        <!-- 01、选择商品类目 -->
        <div class="step-txt">01、{{ $t("product.selectProductCategory") }}</div>
      </div>
      <div class="step-item" :class="{ 'active': postingSteps === 2 }">
        <!-- 02、编辑商品信息 -->
        <div class="step-txt">02、{{ $t("product.editProductInfo") }}</div>
      </div>

    </div>

    <!-- 基本信息  -->
    <div class="title" v-if="postingSteps === 1">
      {{ this.$i18n.t('shopProcess.basicInfo') }}
    </div>

    <!-- <div class="title">新增氢春豆商品</div> -->
    <el-form ref="dataForm" :model="dataForm" label-width="150px" size="small">
      <!-- 选择语言 -->
      <el-form-item v-show="postingSteps === 1" :label="this.$i18n.t('product.selectLanguage')"
        v-if="langItemList.length > 1">
        <el-select v-model="curLang" multiple :placeholder="this.$i18n.t('product.tip1')" class="select-lang"
          @change="selectLang">
          <el-option v-for="item in langItemList" :key="item.lang" :label="item.name" :value="item.lang">
          </el-option>
        </el-select>
        <div class="el-form-item-tips">{{ $t("product.postProductTips2") }}</div>
      </el-form-item>

      <!-- 商品名称 -->
      <div v-show="postingSteps === 1" class="prod-name-box">
        <template v-for="(item, index) in dataForm.prodLangList">
          <el-form-item :label="$t('product.prodName') + (langItemList.length === 1 ? '' : `(${item.langName})`)"
            class="prod-name-con is-required">
            <el-input class="prod-name-input" v-model.trim="item.prodName" maxlength="60" />
            <div class="el-form-item-tips">{{ $t("product.prodNameTip") }}</div>
          </el-form-item>
        </template>
      </div>
      <!-- 商品卖点 -->
      <div v-show="postingSteps === 1" class="prod-name-box">
        <template v-for="(item, index) in dataForm.prodLangList">
          <el-form-item
            :label="$t('product.productSellingPoints') + (langItemList.length === 1 ? '' : `(${item.langName})`)"
            class="prod-name-con">
            <el-input v-model="item.brief" maxlength="100" />
            <div class="el-form-item-tips">{{ $t("product.productSellingPointsTip") }}</div>
          </el-form-item>
        </template>
      </div>

      <el-form-item v-show="postingSteps === 1" :label="this.$i18n.t('product.video')">
        <video-upload v-model="dataForm.video" />
      </el-form-item>
      <el-form-item v-show="postingSteps === 1" :label="this.$i18n.t('discount.commodityPictures')"
        class="productImg-label">
        <!-- <mul-pic-upload v-model="dataForm.imgs" /> -->
        <imgs-upload v-model="dataForm.imgs" />
        <span>{{ $t("platform.recommImgSize") }}800*800，{{ $t("product.picUploadTips") }}</span>
      </el-form-item>

      <!-- 规格库存 -->
      <div class="title" v-if="postingSteps === 1">
        {{ this.$i18n.t('prodSku.specStock') }}
      </div>
      <!-- <sku-tag v-show="postingSteps === 1" ref="skuTag" @change="skuTagChangeSkuHandler"
        :skuList="dataForm.skuList"></sku-tag> -->
      <sku-table v-show="postingSteps === 1" ref="skuTable" v-model="dataForm.skuList" :prodNameCn.sync="prodNameCn"
        :prodNameEn.sync="prodNameEn" :prodLangList='dataForm.prodLangList' :giftListArray="giftListArray"></sku-table>

      <!-- 运费 -->
      <div class="title" v-if="postingSteps === 1">
        {{ this.$i18n.t('freight.shippinngs') }}
      </div>
      <prod-transport v-show="postingSteps === 1" v-model="dataForm.deliveryTemplateId"
        :dataForm="dataForm"></prod-transport>

      <!-- 商品详情 -->
      <el-tabs v-show="postingSteps === 2" type="card">
        <el-tab-pane v-for="(item, index) in dataForm.prodLangList" :key="index" :label="`${item.langName}详情`">
          <el-form-item :label="$i18n.t('homes.productDetails')">
            <tiny-mce v-model="item.content" ref="content" style="width:95%"></tiny-mce>
          </el-form-item>
        </el-tab-pane>
        <!-- <el-tab-pane label="English Information">
          <el-form-item label="Product Details" prop="contentEn">
            <tiny-mce v-model="contentEn" ref="content" style="width:95%"></tiny-mce>
          </el-form-item>
        </el-tab-pane> -->
      </el-tabs>
    </el-form>

    <!-- 底部固定操作栏 -->
    <div class="prod-footer">
      <div class="foot">
        <div class="inner">
          <div v-if="postingSteps === 1">
            <div class="default-btn save-btn" @click="dataFormSubmit">{{ $t("product.saveBtn1") }}</div>
            <div class="default-btn primary-btn" @click="nextStep">{{ $t("product.nextStep2") }}</div>
          </div>
          <div v-if="postingSteps === 2">
            <div @click="prevStep" class="default-btn save-btn">{{ $t("product.prevStep") }}</div>
            <div class="default-btn primary-btn" @click="dataFormSubmit">{{ $t("product.saveBtn1") }}</div>
          </div>
        </div>
      </div>
    </div>
    <!-- 品牌选择弹窗-->
    <brand-select v-if="brandSelectVisible" ref="brandSelect" :isSingle="true"
      @refreshSelectBrand="selectBrand"></brand-select>
  </div>
</template>

<script>
// import MulPicUpload from '@/components/mul-pic-upload'
import VideoUpload from '@/components/video-upload'
import SkuTag from './sku-tag'
import SkuTable from './sku-table'
import TinyMce from '@/components/tiny-mce'
import ImgUpload from '@/components/img-upload'
import ImgsUpload from '@/components/imgs-upload'
import BrandSelect from '@/components/brand-select'
import ProdTransport from './prod-transport'
// import { isHtmlNull } from '@/utils/index.js'
// import { treeDataTranslate, idList } from '@/utils'

export default {
  data() {
    return {
      curLang: [1],
      postingSteps: 1,
      brandSelectVisible: false,
      prodNameEn: '',
      prodNameCn: '',
      sameCityStatus: 0,
      // 规格列表
      dataForm: {
        tableRate: -1, // 0 包邮 -1 固定运费 大于0运费模板id
        prodName: '',
        brief: '',
        pic: '',
        imgs: '',
        video: '',
        categoryId: this.$route.query.categoryId ? parseInt(this.$route.query.categoryId) : null,
        shopCategoryId: null, // 店铺分类id
        prodId: 0,
        brandId: 0,
        prodScore: 0,
        scorePrice: 0,
        skuList: [],
        scoreProdType: 0,
        content: '',
        brandName: '',
        deliveryMode: {},
        isCheck: false,
        value: false,
        deliveryAmount: 0.01, // 统一运费的金额
        deliveryTemplateId: null,
        prodLangList: []
      },
      deliveryMode: {
        hasShopDelivery: true,
        hasUserPickUp: false,
        hasCityDelivery: false
      },
      brandName: '',
      isSubmit: false,
      resourcesUrl: process.env.VUE_APP_RESOURCES_URL,
      // 语言列表
      langItemList: [],
      masterLangInfo: { name: '', lang: '' },
      giftListArray: [],// 礼品券数组
    }
  },
  watch: {
    'dataForm.imgs': {
      deep: true, // 深度监听设置为 true
      immediate: true,
      handler(newV, oldV) {
        if (newV.split(',').length > 9) {
          this.dataForm.imgs = oldV
          this.errorMsg(this.$i18n.t('product.downloadTemplateTips3'))
        }
      }
    },
    'dataForm.prodLangList': {
      deep: true, // 深度监听设置为 true
      immediate: true,
      handler() {
        this.skuTagChangeSkuHandler(this.dataForm.skuList)
      }
    }
  },
  components: {
    // MulPicUpload,
    BrandSelect,
    ImgUpload,
    ImgsUpload,
    VideoUpload,
    TinyMce,
    SkuTag,
    SkuTable,
    ProdTransport
  },
  computed: {
    defalutSku() {
      return this.$store.state.prod.defalutSku
    }
  },
  mounted() {
    this.dataForm.prodId = this.$route.query.prodId
    // this.getCategoryList()
    // this.getCategoryTree()
    this.getDataList()
    this.getGiftList()
  },
  methods: {
    // 获取礼品券接口
    getGiftList() {
      this.$http({
        url: this.$http.adornUrl('/platform/search/prod/get/list'),
        params: this.$http.adornParams(
          Object.assign({ current: 1, size: 10000 }), false
        )
      }).then(({ data }) => {
        this.giftListArray = data.records
      })
    },
    getLangList() {
      this.$http({
        url: this.$http.adornUrl('/sys/lang'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({ data }) => {
        if (data) {
          const info = data
          this.masterLangInfo.name = info.name
          this.masterLangInfo.lang = info.lang
          this.langItemList = info.langItemList

          if (!this.dataForm.prodLangList.length) {
            this.selectLang([info.lang])
          } else {
            let curLang = []
            let masterInfo = null
            // 将主语言信息移到第一个
            for (let i = 0; i < this.dataForm.prodLangList.length; i++) {
              const prodLang = this.dataForm.prodLangList[i]
              if (i !== 0 && prodLang.lang === info.lang) {
                masterInfo = prodLang
                this.dataForm.prodLangList.splice(i, 1)
                break
              }
            }
            if (masterInfo) {
              this.dataForm.prodLangList.unshift(masterInfo)
            }
            // 设置langName并且获取当前语言的信息
            this.dataForm.prodLangList.forEach((item, index) => {
              const fd = this.langItemList.find(f => f.lang === item.lang)
              if (fd) {
                item.langName = fd ? fd.name : ''
                curLang.push(item.lang)
              }
            })
            this.selectLang(curLang)
          }
        }
      })
    },
    // 上一步
    prevStep() {
      this.postingSteps = 1
    },
    // 下一步
    nextStep() {
      this.$refs['dataForm'].validate((valid) => {
        const langList = this.dataForm.prodLangList
        for (const item of langList) {
          if (!item.prodName) {
            this.errorMsg(this.$i18n.t('product.pleComProdName'))
            return
          }
        }
        // 运费处理
        this.dataForm.deliveryTemplateId = this.dataForm.tableRate < 1 ? this.dataForm.tableRate : this.dataForm.deliveryTemplateId
        // 运费处理
        if (this.dataForm.deliveryTemplateId !== -1) {
          this.dataForm.deliveryAmount = null
        }
        if (!this.dataForm.imgs) {
          this.errorMsg(this.$i18n.t('product.choosePicUpload'))
          return
        }
        if (!this.dataForm.skuList.find(el => el.status)) {
          // 至少要启用一种商品规格
          this.$message({
            message: this.$i18n.t('product.enableSpec'),
            type: 'error',
            duration: 1000
          })
          return
        }
        // 检验是否有选择品牌
        // if (!this.dataForm.brandName) {
        //   this.errorMsg('请选择一个所属品牌')
        //   return
        // }

        this.dataForm.skuList.forEach((e) => {
          if (e.giftList2) {
            e.giftList = e.giftList2.toString()
          } else {
            e.giftList = ''
          }
        })

        // 校验sku列表
        this.checkSkuList()
        if (this.isCheck) {
          this.errorMsg(this.value)
          return
        }

        // if (!this.prodNameEn) {
        //   this.prodNameEn = this.prodNameCn
        // }
        // if (/^\s+$/g.test(this.prodNameEn)) {
        //   this.prodNameEn = this.prodNameCn
        // }
        // if (/^\s+$/g.test(this.briefEn) || /^\s+$/g.test(this.briefCn)) {
        //   this.errorMsg(this.$i18n.t('shopProcess.inputAllSpace'))
        //   return
        // }
        if (this.dataForm.deliveryTemplateId === null) {
          this.errorMsg(this.$i18n.t('product.pleShgTlate'))
          return
        }
        this.postingSteps = 2
      })
    },
    /**
 * 选择语言(中文信息必填)
 */
    selectLang(value) {
      this.curLang = JSON.parse(JSON.stringify(value))
      if (!value.length) {
        this.curLang = [this.masterLangInfo.lang]
        return
      }
      // 设置主语言不可删除
      if (!this.curLang.includes(this.masterLangInfo.lang)) {
        this.curLang.unshift(this.masterLangInfo.lang)
      }

      // 所选语言改变
      const tempArr = this.dataForm.prodLangList.filter(item => this.curLang.includes(item.lang))

      this.curLang.forEach((item, index) => {
        if (!tempArr.find(f => f.lang == item)) {
          const fd = this.langItemList.find(it => it.lang === item)
          if (fd) {
            tempArr.splice(index, 0, { langName: fd.name, brief: '', content: '', lang: item, prodId: this.dataForm.prodId, prodName: '' })
          }
        }
      })
      this.dataForm.prodLangList = tempArr

      // sku列表变化
      const unSelLang = this.langItemList.map(m => !this.curLang.includes(m.lang))
      this.dataForm.skuList.forEach(sku => {
        for (const lang of unSelLang) {
          sku[`prodName_${lang}`] = ''
        }
      })
    },
    // 获取分类数据
    getDataList() {
      this.isSubmit = false
      if (this.dataForm.prodId) {
        // 获取产品数据
        this.$http({
          url: this.$http.adornUrl(`/platform/scoreProduct/info/${this.dataForm.prodId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({ data }) => {
          let data2 = data
          data2.skuList.forEach(e => {
            e.giftList2 = e.giftList ? e.giftList.split(',').map(Number) : []
          })
          this.dataForm = data2
          // 获取语言列表
          this.getLangList()

          // 运费选项处理
          if (this.dataForm.deliveryTemplateId < 1) {
            // this.dataForm.tableRate = this.dataForm.deliveryTemplateId
            this.$set(this.dataForm, 'tableRate', this.dataForm.deliveryTemplateId)
            this.dataForm.deliveryTemplateId = null
          } else {
            // this.dataForm.tableRate = 1
            this.$set(this.dataForm, 'tableRate', 1)
          }
          this.dataForm.deliveryMode = JSON.parse(data.deliveryMode)
          this.dataForm.deliveryMode.hasShopDelivery = true
          if (data.brand) {
            this.dataForm.brandId = data.brand.brandId
            this.dataForm.brandName = data.brand.brandName
            this.brandName = data.brandName
          }

          this.dataForm.skuList.forEach(sku => {
            sku.oriStock = sku.stocks
            sku.changeStock = sku.stocks
            if (sku.skuLangList) {
              sku.skuLangList.forEach(skulang => {
                this.$set(sku, `prodName_${skulang.lang}`, skulang.prodName)
              })
            }
          })
          this.$refs.skuTag.init(data.skuList)
          this.$refs.skuTable.init(this.dataForm.skuList)
        })
      } else {
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          this.$refs.skuTag.init()
          this.dataForm.pic = ''
          this.dataForm.imgs = ''
          this.dataForm.video = ''
        })
        // 获取语言列表
        this.getLangList()
      }
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate((valid) => {
        const langList = this.dataForm.prodLangList
        for (const item of langList) {
          if (!item.prodName) {
            this.errorMsg(this.$i18n.t('product.pleComProdName'))
            return
          }
        }
        // 运费处理
        this.dataForm.deliveryTemplateId = this.dataForm.tableRate < 1 ? this.dataForm.tableRate : this.dataForm.deliveryTemplateId
        // 运费处理
        if (this.dataForm.deliveryTemplateId !== -1) {
          this.dataForm.deliveryAmount = null
        }
        if (!this.dataForm.imgs) {
          this.errorMsg(this.$i18n.t('product.choosePicUpload'))
          return
        }
        if (!this.dataForm.skuList.find(el => el.status)) {
          // 至少要启用一种商品规格
          this.$message({
            message: this.$i18n.t('product.enableSpec'),
            type: 'error',
            duration: 1000
          })
          return
        }
        // 检验是否有选择品牌
        // if (!this.dataForm.brandName) {
        //   this.errorMsg('请选择一个所属品牌')
        //   return
        // } if (e.giftList2) {
        this.dataForm.skuList.forEach((e) => {
          if (e.giftList2) {
            e.giftList = e.giftList2.toString()
          } else {
            e.giftList = ''
          }
        })

        // 校验sku列表
        this.checkSkuList()
        if (this.isCheck) {
          this.errorMsg(this.value)
          return
        }
        // if (/^\s+$/g.test(this.briefEn) || /^\s+$/g.test(this.briefCn)) {
        //   this.errorMsg(this.$i18n.t('shopProcess.inputAllSpace'))
        //   return
        // }
        if (this.dataForm.deliveryTemplateId === null) {
          this.errorMsg(this.$i18n.t('product.pleShgTlate'))
          return
        }
        this.dataForm.prodName = this.dataForm.prodLangList[0].prodName || ''
        let param = Object.assign({}, this.dataForm)

        // 设置价格和库存
        this.paramSetPriceAndStocks(param)
        param.deliveryMode = undefined
        param.deliveryModeVo = this.deliveryMode
        this.dataForm.skuList.forEach(item => {
          item.changeStock = item.stocks - item.oriStock
        })
        // 商品主图
        param.pic = this.dataForm.imgs.split(',')[0]
        if (this.isSubmit) {
          return false
        }
        // 设置不同语言的sku名称
        this.dataForm.skuList.forEach(sku => {
          const skuLangList = []
          sku.prodName = sku[`prodName_${this.masterLangInfo.lang}`]
          let properties = sku.properties.split(';')
          for (const propertiesKey in properties) {
            sku.skuName += properties[propertiesKey].substring(properties[propertiesKey].indexOf(':') + 1) + ' '
          }
          for (const lang of this.curLang) {
            skuLangList.push({
              lang,
              prodName: sku[`prodName_${lang}`],
              properties: sku.properties,
              skuId: sku.skuId,
              skuName: sku.skuName
            })
          }
          sku.skuLangList = skuLangList
        })
        // if (1) {
        //   return
        // }

        this.isSubmit = true
        this.$http({
          url: this.$http.adornUrl(`/platform/scoreProduct`),
          method: param.prodId ? 'put' : 'post',
          data: this.$http.adornData(param)
        }).then(({ data }) => {
          this.$message({
            message: this.$i18n.t('remindPop.success'),
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.visible = false
              this.$store.commit('common/removeMainActiveTab')
              this.$router.push({ name: 'prod-scoreProdList' })
              this.$emit('refreshDataList')
            }
          })
        }).catch((e) => {
          this.isSubmit = false
        })
      })
    },
    /**
     * 品牌删除按钮
     */
    handleClose() {
      this.dataForm.brandId = null
      this.brandName = ''
      this.dataForm.brandName = this.brandName
    },
    /**
     * 显示添加指定品牌弹框
     */
    brandSelectHandle() {
      this.brandSelectVisible = true
      this.$nextTick(() => {
        this.$refs.brandSelect.init()
      })
    },
    /**
     * 添加指定品牌
     */
    selectBrand(brands) {
      if (brands) {
        this.brandName = brands[0].brandName
        this.dataForm.brandId = brands[0].brandId
        this.dataForm.brandName = this.brandName
        // this.$set(this.dataForm, 'brandId', this.dataForm.brandId)
        // this.$set(this.dataForm, 'brandName', this.dataForm.brandName)
      }
    },
    /**
     * 删除视频
     */
    deleteVideo() {
      this.dataForm.video = null
    },

    checkSkuList() {
      // console.log(this.dataForm.skuList)
      this.dataForm.skuList.forEach(item => {
        this.isCheck = false
        // if (!item.pic) {
        //   this.isCheck = true
        //   this.value = '请上传sku图片'
        //   return false
        // }
        if (!item.price && item.price !== 0) {
          this.isCheck = true
          this.value = this.$i18n.t('product.emptyPrice')
          return false
        }
        if (!item.oriPrice && item.oriPrice !== 0) {
          this.isCheck = true
          this.value = this.$i18n.t('product.emptyMarketValue')
          return false
        }
        // if (!item.skuScore) {
        //   this.isCheck = true
        //   this.value = this.$i18n.t('product.emptyScorePrice')
        //   return false
        // }
        if (item.stocks === null || item.stocks === undefined) {
          this.isCheck = true
          this.value = this.$i18n.t('product.emptyStocks')
          return false
        }
        if (item?.giftList2?.length && !item.giftNumber) {
          this.isCheck = true
          this.value = '礼品券数量不能为空'
          return false
        }
        if (!item?.giftList2?.length && item.giftNumber) {
          this.isCheck = true
          this.value = '请选择礼品券'
          return false
        }
      })
    },
    paramSetPriceAndStocks(param) {
      // 获取规格属性信息
      // param.skuList = this.$refs.prodSpec.getTableSpecData()
      // 商品库存
      param.totalStocks = 0
      // 商品价格
      param.price = param.skuList[0].price
      // 商品原价
      param.oriPrice = ''
      // 商品氢春豆价格
      param.scorePrice = param.skuList[0].skuScore
      // 商品实际库存
      for (let i = 0; i < param.skuList.length; i++) {
        const element = param.skuList[i]
        if (element.status !== 1) {
          continue
        }
        if (i === 0) {
          param.price = element.price ? Number.parseFloat(element.price) : 0
          param.scorePrice = element.skuScore ? Number.parseFloat(element.skuScore) : 0
          param.oriPrice = element.oriPrice ? Number.parseFloat(element.oriPrice) : 0
        }
        // 商品价格为最低价的那件商品的价格
        param.price = Math.min(param.price, element.price)
        if (param.price === element.price) {
          param.price = element.price
        }
        // 设置最低市场价
        param.oriPrice = param.oriPrice !== '' ? Math.min(param.oriPrice, element.oriPrice) : element.oriPrice
        if (param.oriPrice === element.oriPrice) {
          param.oriPrice = element.oriPrice
        }
        // 设置最低氢春豆价格
        param.scorePrice = Math.min(param.scorePrice, element.skuScore)
        if (param.scorePrice === element.skuScore) {
          param.scorePrice = element.skuScore
        }

        param.totalStocks += element.stocks ? Number.parseInt(element.stocks) : 0
      }
      // 如果sku没有商品名称，则使用商品的商品名称
      // if (param.skuList.length === 1 && !param.skuList[0].skuName) {
      //   param.skuList[0].prodNameCn = this.dataForm.prodNameCn
      //   param.skuList[0].prodNameEn = this.dataForm.prodNameEn
      // }
    },
    skuTagChangeSkuHandler(skuList) {
      // const prodNameCn = this.prodNameCn
      // const prodNameEn = this.prodNameEn
      const prodName = this.dataForm.prodLangList[0] ? this.dataForm.prodLangList[0].prodName : ''
      skuList.forEach(sku => {
        if (sku.properties) {
          sku.skuName = ''
          let properties = sku.properties.split(';')
          for (const propertiesKey in properties) {
            sku.skuName += properties[propertiesKey].substring(properties[propertiesKey].indexOf(':') + 1) + ' '
          }
          sku.prodName = prodName + ' ' + sku.skuName
          for (const item of this.dataForm.prodLangList) {
            this.$set(sku, [`prodName_${item.lang}`], item.prodName + ' ' + sku.skuName)
          }
          // sku.prodNameCn = prodNameCn + ' ' + sku.skuName
          // sku.prodNameEn = prodNameEn + ' ' + sku.skuName
        }
      })
      this.dataForm.skuList = skuList
    },
    errorMsg(message) {
      this.$message({
        message: message,
        type: 'error',
        duration: 1500
      })
    }
  }
}
</script>
<style>
.el-upload--picture-card {
  background: #ffffff;
}

.productImg-label .el-form-item__label:before {
  content: '*';
  color: #F56C6C;
  margin-right: 4px;
}

.productImg-label .el-upload--picture-card i {
  position: absolute;
  top: 39%;
  left: 37%;
}

/* .stress {
          color: #FF2120;
          padding-right: 5px;
        } */

.mod-prod-info .el-textarea__inner:focus {
  border-color: #c0c4cc !important;
}
</style>
<style lang="scss" scoped>
.mod-prod-info {

  // 步骤
  .post-step {
    margin-bottom: 30px;
    display: flex;
    align-content: center;
    justify-content: space-between;

    .step-item {
      position: relative;
      flex: 1;

      .step-txt {
        display: block;
        font-size: 14px;
        text-align: center;
        box-sizing: border-box;
        background: #f9f9f9;
        padding: 12px 0;
      }
    }

    .step-item.active {
      .step-txt {
        color: #fff;
        background: #155bd4;
      }
    }

    // 右箭头
    .step-item:not(:last-child) {
      .step-txt {
        margin-right: 10px;
      }

      &::after {
        position: absolute;
        top: 0;
        right: 0;
        content: '';
        width: 0;
        height: 0;
        border-top: 20px solid transparent;
        border-left: 10px solid #f9f9f9;
        border-bottom: 20px solid transparent;
      }
    }

    .step-item.active:not(:last-child) {
      &::after {
        border-left: 10px solid #155bd4;
      }
    }

    // 左箭头
    .step-item:not(:first-child) {
      &::before {
        position: absolute;
        top: 0;
        left: 0;
        content: '';
        width: 0;
        height: 0;
        border-top: 20px solid transparent;
        border-left: 10px solid #fff;
        border-bottom: 20px solid transparent;
      }
    }
  }

  /* 每项内容的标题 */
  .title {
    margin: 5px 0 28px 0px;
    padding-left: 15px;
    background-color: #f6f6f6;
    line-height: 40px;
    font-weight: 800;
  }

  // 底部固定操作栏
  .prod-footer {
    width: calc(100% - 250px);
    left: 250px;
    position: fixed;
    bottom: 0;
    width: 93%;
    box-shadow: 0 -3px 5px #eee;
    z-index: 3;
    margin-top: 20px;
    margin-right: 20px;

    .foot {
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 10px 0;
      background: #fff;
      box-sizing: border-box;

      .inner {
        .default-btn.save-btn {
          border-color: #155bd4;
          color: #155bd4;
        }
      }
    }
  }

  // 选择语言
  .select-lang {
    display: block;
    width: 400px;
  }

  & ::v-deep .el-input__inner {
    color: #303133;
  }

  // 商品名称
  .prod-name-box {
    &>>>.el-input__inner {
      width: 400px;
      padding: 0 8px !important;
    }

    .prod-name-con {
      display: inline-block;
      margin-right: 15px;
    }

    .prod-name-input {
      width: 400px;
    }
  }
}
</style>
<style scoped>
.mod-prod-info /deep/ .el-form-item__label {
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
