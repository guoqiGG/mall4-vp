<template>
  <div class="mod-prod-sku-table">
    <el-form-item
      :label="this.$i18n.t('prodSku.priceAndInventory')"
      :label-width="this.$i18n.t('language') === 'English' ? '150px' : '130px'"
    >
      <div class="sku-table-con">
        <!--sku列表-->
        <div class="table-con">
          <el-table
            :data="lists"
            header-cell-class-name="table-header"
            :span-method="tableSpanMethod"
            row-class-name="table-row"
            border
          >
            <el-table-column
              v-for="(leftTitle, index) in tableLeftTitles"
              :key="index"
              :label="
                $t('language') === 'English'
                  ? leftTitle.tagNameEn
                  : leftTitle.tagName
              "
            >
            <!-- scope.row.propertiesEn &&  -->
              <template slot-scope="scope">
                <div v-if="scope.row.properties">
                  {{
                    scope.row.properties.split(";")[index].substring(scope.row.properties.split(";")[index].indexOf(':') + 1)
                  }}
                </div>
              </template>
            </el-table-column>
            <!-- 市场价 -->
            <el-table-column
              prop="oriPrice"
              :label="this.$i18n.t('prodList.marketValue')"
            >
              <template slot-scope="scope">
                <input
                  v-model.number="scope.row.oriPrice"
                  type="number"
                  :precision="2"
                  :max="100000000"
                  :min="0"
                  :step="0.01"
                  :disabled="!scope.row.status"
                  class="tag-input-width"
                  @blur="
                    handleInputValue(
                      scope.row.oriPrice,
                      scope.$index,
                      'oriPrice',
                      0,
                      100000000
                    )
                  "
                />
              </template>
            </el-table-column>
            <!-- 售价 -->
            <el-table-column
              prop="price"
              :label="this.$i18n.t('prodList.salesPrice')"
            >
              <template slot-scope="scope">
                <input
                  v-model.number="scope.row.price"
                  type="number"
                  :precision="2"
                  :max="100000000"
                  :min="0.01"
                  :step="0.01"
                  :disabled="!scope.row.status"
                  class="tag-input-width"
                  @blur="
                    handleInputValue(
                      scope.row.price,
                      scope.$index,
                      'price',
                      0.01,
                      100000000
                    )
                  "
                />
              </template>
            </el-table-column>
            <!-- 积分价 -->
            <el-table-column
              prop="skuScore"
              :label="this.$i18n.t('product.scorePrice')"
            >
              <template slot-scope="scope">
                <input
                  v-model.number="scope.row.skuScore"
                  type="number"
                  :precision="2"
                  :max="100000000"
                  :min="1"
                  :step="1"
                  :disabled="!scope.row.status"
                  class="tag-input-width"
                  @blur="
                    handleInputValue(
                      scope.row.skuScore,
                      scope.$index,
                      'skuScore',
                      1,
                      100000000
                    )
                  "
                />
              </template>
            </el-table-column>
            <!-- 库存 -->
            <el-table-column
              prop="stocks"
              :label="this.$i18n.t('product.stocks')"
            >
              <template slot-scope="scope">
                <input
                  v-model.number="scope.row.stocks"
                  type="number"
                  :max="9999999"
                  :min="0"
                  :step="1"
                  :disabled="!scope.row.status"
                  class="tag-input-width"
                  @keyup="
                    scope.row.stocks = String(scope.row.stocks).match(/[^0-9]/)
                      ? ''
                      : scope.row.stocks
                  "
                  @blur="
                    handleInputValue(
                      scope.row.stocks,
                      scope.$index,
                      'stocks',
                      0,
                      9999999
                    )
                  "
                />
              </template>
            </el-table-column>
            <!-- 重量 -->
            <el-table-column
              prop="weight"
              :label="this.$i18n.t('prodList.prodWeight')"
            >
              <template slot-scope="scope">
                <input
                  v-model.number="scope.row.weight"
                  type="number"
                  :max="100000000"
                  :min="0"
                  :step="0.01"
                  :disabled="!scope.row.status"
                  class="tag-input-width"
                  @blur="
                    handleInputValue(
                      scope.row.weight,
                      scope.$index,
                      'weight',
                      0,
                      100000000
                    )
                  "
                />
              </template>
            </el-table-column>
            <!-- 体积 -->
            <el-table-column
              prop="volume"
              :label="this.$i18n.t('prodList.prodVolume')"
            >
              <template slot-scope="scope">
                <input
                  v-model.number="scope.row.volume"
                  type="number"
                  :max="100000000"
                  :min="0"
                  :step="0.01"
                  :disabled="!scope.row.status"
                  class="tag-input-width"
                  @blur="
                    handleInputValue(
                      scope.row.volume,
                      scope.$index,
                      'volume',
                      0,
                      100000000
                    )
                  "
                />
              </template>
            </el-table-column>
            <!-- 编码 -->
            <el-table-column
              prop="partyCode"
              :label="this.$i18n.t('product.prodCode')"
              width="220px"
            >
              <template slot-scope="scope">
                <input
                  ref="partyCodeInt"
                  v-model="scope.row.partyCode"
                  type="text"
                  maxlength="36"
                  :disabled="!scope.row.status"
                  class="tag-input-width text-input party-code"
                  @blur="validatePartyCode(scope)"
                />
              </template>
            </el-table-column>
            <el-table-column
              fixed="right"
              :label="this.$i18n.t('text.menu')"
              align="center"
            >
              <!-- :width="this.$i18n.t('language') === 'English' ? '210' : '120'" -->
              <template slot-scope="scope">
                <div
                  class="default-btn text-btn"
                  @click="changeSkuStatus(`${scope.$index}`)"
                  v-if="scope.row.status"
                >
                  {{ $t("publics.disable") }}
                </div>
                <div
                  class="default-btn text-btn"
                  @click="changeSkuStatus(`${scope.$index}`)"
                  v-else
                >
                  {{ $t("shop.ena") }}
                </div>
              </template>
            </el-table-column>
          </el-table>
        </div>

        <!--批量设置-->
        <div class="batch-settings-box">
          <div class="set-txt">
            <span class="default-btn text-btn" @click="switchSet">{{
                $t("groups.batchSettings")
              }}</span>
            <span class="weak-txt">{{ $t("prodSku.postProductTips14") }}</span>
          </div>
          <div v-if="isEdit" class="batch-settings-tb">
            <!-- 头部 -->
            <ul class="batch-settings-Head">
              <li
                class="head-item"
                v-for="(item, i) in tableLeftTitles"
                :key="i"
              >
                {{item.tagName}}
              </li>
              <li class="head-item">{{ $t("prodList.marketValue") }}</li>
              <li class="head-item">{{ $t("prodList.salesPrice") }}</li>
              <li class="head-item">{{ $t("product.scorePrice") }}</li>
              <li class="head-item">
                {{ $t("product.stocks") }}
              </li>
              <li class="head-item">
                {{ $t("prodList.prodWeight") }}
              </li>
              <li class="head-item">
                {{$t("prodList.prodVolume")}}
              </li>
              <li class="coding"></li>
              <li class="head-item"></li>

              <!-- <li class="head-item">{{ $t('product.commodityCode') }}</li> -->
            </ul>
            <el-form @submit.native.prevent :inline="true" class="demo-form-inline">
              <div class="batch-settings-con">
                <div class="item" v-for="(item, i) in batchList" :key="i">
                  <el-select
                    v-model="item.value"
                    size="small"
                    class="bat-set-item"
                    :placeholder="$t('tip.select')"
                    @change="changeValue"
                  >
                    <el-option
                      :key="-1"
                      :label="$t('date.a')"
                      :value="-1"
                    />
                    <el-option
                      v-for="(el,index) in item.tagItems"
                      :key="index"
                      :label="el.propValue"
                      :value="el.propValue"
                    />
                  </el-select>
                </div>

                <div class="item">
                  <input
                    v-model.number="dataFrom.oriPrice"
                    type="number"
                    :precision="2"
                    :max="100000000"
                    :min="0"
                    :step="0.01"
                    class="tag-input-width"
                    @blur="
                      handleInputValue(
                        dataFrom.oriPrice,
                        null,
                        'oriPrice',
                        0,
                        100000000
                      )
                    "
                  />
                </div>
                <div class="item">
                  <input
                    v-model.number="dataFrom.price"
                    type="number"
                    :precision="2"
                    :max="100000000"
                    :min="0.01"
                    :step="0.01"
                    class="tag-input-width"
                    @blur="
                      handleInputValue(
                        dataFrom.price,
                        null,
                        'price',
                        0.01,
                        100000000
                      )
                    "
                  />
                </div>
                <div class="item">
                  <input
                    v-model.number="dataFrom.skuScore"
                    type="number"
                    :precision="2"
                    :max="100000000"
                    :min="1"
                    :step="1"
                    class="tag-input-width"
                    @blur="
                      handleInputValue(
                        dataFrom.skuScore,
                        null,
                        'skuScore',
                        1,
                        100000000
                      )
                    "
                  />
                </div>
                <div class="item">
                  <input
                    v-model.number="dataFrom.stocks"
                    type="number"
                    :max="9999999"
                    :min="0"
                    :step="1"
                    class="tag-input-width"
                    @keyup="
                      dataFrom.stocks = String(dataFrom.stocks).match(/[^0-9]/)
                        ? ''
                        : dataFrom.stocks
                    "
                    @blur="
                      handleInputValue(
                        dataFrom.stocks,
                        null,
                        'stocks',
                        0,
                        9999999
                      )
                    "
                  />
                </div>
                <div class="item">
                  <input
                    v-model.number="dataFrom.weight"
                    type="number"
                    :max="100000000"
                    :min="0"
                    :step="0.01"
                    class="tag-input-width"
                    @blur="
                      handleInputValue(
                        dataFrom.weight,
                        null,
                        'weight',
                        0,
                        100000000
                      )
                    "
                  />
                </div>
                <div class="item">
                  <input
                    v-model.number="dataFrom.volume"
                    type="number"
                    :max="100000000"
                    :min="0"
                    :step="0.01"
                    class="tag-input-width"
                    @blur="
                      handleInputValue(
                        dataFrom.volume,
                        null,
                        'volume',
                        0,
                        100000000
                      )
                    "
                  />
                </div>
                <div class="coding"></div>
                <div class="item"></div>


              </div>
              <div class="btn-row">
                <div class="default-btn primary-btn" @click="batchSet">
                  {{ $t("crud.saveBtn") + $t("groups.batchSettings") }}
                </div>
                <div class="default-btn" @click="switchSet">
                  {{ $t("crud.filter.cancelBtn") }}
                </div>
              </div>

            </el-form>
          </div>
        </div>
      </div>
    </el-form-item>
  </div>
</template>

<script>
// import PicUpload from '@/components/pic-upload'
import { validNoEmptySpace } from '@/utils/validate'
import ImgUpload from '@/components/img-upload'
import { flatten as genFlatten } from '@/utils'
import Big from 'big.js'

export default {
  data () {
    return {
      batchList: [],
      // 表格数据
      lists: [],
      rowspan: [],
      // 根据选定的规格所查询出来的规格值
      dbSpecValues: [],
      specs: [], // 使用的规格
      // initing: false,
      isEdit: false,
      dataFrom: {
        oriPrice: null,
        price: null,
        skuScore: null,
        stocks: null,
        weight: null,
        volume: null
      },
      partyCodeErrTips: false
    }
  },
  components: {
    ImgUpload
  },
  props: {
    value: {
      default: [],
      type: Array
    },
    prodNameCn: {
      default: ''
    },
    prodNameEn: {
      default: ''
    }
  },
  watch: {
    lists: {
      deep: true,
      immediate: true,
      handler (val) {
        
      }
    },
    skuTags: {
      deep: true,
      immediate: true,
      handler () {
        this.lists = genFlatten(this.skuTags, this.lists, this.defalutSku)
        this.computeRowspan()
      }
    },
    value (val) {},
    prodNameCn: function () {
      this.skuAddProdName()
    },
    prodNameEn: function () {
      this.skuAddProdName()
    },
    rowSpan: function () {}
  },
  created: function () {
    this.isEdit = false
    if (!this.lists.length) {
      this.lists.push({
        oriPrice: 0.01,
        partyCode: '',
        price: 0.01,
        prodName: '',
        status: 1,
        stocks: 0,
        volume: 0,
        weight: 0,
        skuScore: 0
      })
    }
  },
  computed: {
    tableLeftTitles () {
      this.batchList = JSON.parse(JSON.stringify(this.skuTags))
      this.batchList.forEach(el => {
        if (!el.value) {
          el.value = -1
        }
      })
      return this.skuTags
    },
    skuTags: {
      get () {
        return this.$store.state.prod.skuTags.filter((item) => {
          return !!(
            item.tagItems &&
            item.tagItems.length &&
            !item.tagItems[0].creating
          )
        })
      }
    },
    defalutSku () {
      return this.$store.state.prod.defalutSku
    }
  },
  methods: {
    init (skuList) {
      // this.initing = true
      this.lists = genFlatten(this.skuTags, skuList, this.defalutSku)
      this.computeRowspan()
    },
    getTableSpecData () {
      return this.lists
    },
    getDataList () {
      return this.lists
    },
    changeSkuImg (propValue, img) {
      // 把对应的sku图片修改
      for (let i = 0; i < this.lists.length; i++) {
        if (!this.lists[i].properties) {
          continue
        }
        let pVal = this.lists[i].properties.split(';')[0]
        let properties = pVal.substring(pVal.indexOf(':') + 1)
        if (properties === propValue) {
          this.lists[i].pic = img
        }
      }
    },
    clearSkuImg () {
      for (let i = 0; i < this.lists.length; i++) {
        this.lists[i].pic = ''
      }
    },
    computeRowspan () {
      this.rowspan = []
      const rowspan = (index) => {
        let span = []
        let dot = 0
        this.lists.map((item, idx) => {
          if (idx === 0) {
            span.push(1)
          } else {
            if (
              this.checkIsEqualByIndex(
                item.properties,
                this.lists[idx - 1].properties,
                index
              )
            ) {
              span[dot] += 1
              span.push(0)
            } else {
              dot = idx
              span.push(1)
            }
          }
        })

        this.rowspan.push(span)
      }
      this.skuTags.map((item, index) => {
        rowspan(index)
      })
    },
    checkIsEqualByIndex (str1, str2, index, splitStr = ':') {
      let strArr1 = str1.split(';')
      let strArr2 = str2.split(';')
      let temp1 = [strArr1[index].slice(0, strArr1[index].indexOf(':')), strArr1[index].substring(strArr1[index].indexOf(':') + 1)]
      let temp2 = [strArr2[index].slice(0, strArr2[index].indexOf(':')), strArr2[index].substring(strArr2[index].indexOf(':') + 1)]
      return temp1[1] === temp2[1]
    },
    tableSpanMethod ({ row, column, rowIndex, columnIndex }) {
      for (let i = 0; i < this.skuTags.length; i++) {
        if (columnIndex === i) {
          if (this.rowspan[i] && this.rowspan[i][rowIndex]) {
            return {
              rowspan: this.rowspan[i][rowIndex],
              colspan: 1
            }
          } else {
            return {
              rowspan: 0,
              colspan: 0
            }
          }
        }
      }
    },
    changeSkuStatus (tagIndex) {
      this.lists[tagIndex].status = this.lists[tagIndex].status ? 0 : 1
      this.$emit('input', this.lists)
    },
    skuAddProdName () {
      // if (this.initing) {
      //   return
      // }
      let skuList = []
      this.$emit('input', this.lists)
      for (let i = 0; i < this.lists.length; i++) {
        const sku = Object.assign({}, this.lists[i])
        if (!sku.properties) {
          return
        }
        sku.skuName = ''
        sku.skuNameEn = ''
        let properties = sku.properties.split(';')
        for (const propertiesKey in properties) {
          sku.skuName += properties[propertiesKey].substring(properties[propertiesKey].indexOf(':') + 1) + ' '
        }
        sku.prodNameCn = this.prodNameCn + ' ' + sku.skuName
        sku.prodName = this.prodNameCn + ' ' + sku.skuName
        sku.prodNameEn = this.prodNameEn + ' ' + sku.skuNameEn
        skuList.push(sku)
      }
      this.$emit('input', skuList)
    },
    // 清除已选择的单品
    clearSingleProds () {
      for (let i = 0; i < this.lists.length; i++) {
        this.lists[i].skuSingleProds = []
      }
      this.$emit('input', this.lists)
    },
    // 清除所有sku的库存
    clearStocks () {
      for (let i = 0; i < this.lists.length; i++) {
        this.lists[i].stocks = 0
      }
      this.$emit('input', this.lists)
    },
    switchSet () {
      this.isEdit = !this.isEdit
      if (!this.isEdit) {
        this.dataFrom.oriPrice = null
        this.dataFrom.price = null
        this.dataFrom.stocks = null
        this.dataFrom.weight = null
        this.dataFrom.volume = null
        this.dataFrom.skuScore = null
        this.dataFrom.partyCode = ''
      }
    },
    changeValue () {
      this.$forceUpdate() // 刷新
    },
    batchSet () {
      this.lists.forEach((sku) => {
        let isBatch = true
        sku.properties.split(';').forEach((el, index) => {
          // -1为全部
          if (this.batchList[index].value !== -1 && el !== this.batchList[index].tagName + ':' + this.batchList[index].value) {
            isBatch = false
          }
        })
        if (isBatch) {
          if (this.dataFrom.oriPrice || this.dataFrom.oriPrice === 0) {
            sku.oriPrice = this.dataFrom.oriPrice
          }
          if (this.dataFrom.price) {
            sku.price = this.dataFrom.price
          }
          if (this.dataFrom.stocks || this.dataFrom.stocks === 0) {
            sku.stocks = this.dataFrom.stocks
            // this.initing = false
          }
          if (this.dataFrom.weight || this.dataFrom.weight === 0) {
            sku.weight = this.dataFrom.weight
          }
          if (this.dataFrom.volume || this.dataFrom.volume === 0) {
            sku.volume = this.dataFrom.volume
          }
          if (this.dataFrom.skuScore || this.dataFrom.skuScore === 0) {
            sku.skuScore = this.dataFrom.skuScore
          }
        }
      })
      // this.switchSet()
      // this.isEdit = false
      this.skuAddProdName()
    },

    /**
     * 处理输入框数据
     * @param data
     * @param index
     * @param dataFields
     * @param min 最小值
     * @param max 最大值
     */
    handleInputValue (data, index, dataFields, min, max) {
      if (index !== undefined && index !== null) {
        // 表格
        if (+data > max) {
          this.$set(this.lists[index], dataFields, max)
          if (dataFields === 'stocks') {
            this.$emit('input', this.lists)
          }
          return
        }
        if (+data <= min || !data) {
          this.$set(this.lists[index], dataFields, min)
          if (dataFields === 'stocks') {
            this.$emit('input', this.lists)
          }
          return
        }
        if (
          dataFields === 'price' ||
          dataFields === 'oriPrice' ||
          dataFields === 'volume' ||
          dataFields === 'weight' ||
          dataFields === 'skuScore'
        ) {
          this.$set(this.lists[index], dataFields, this.checkInput(data))
        }
      } else {
        // 批量
        if (data === null || data === '') {
          return
        }
        if (+data > max) {
          this.$set(this.dataFrom, dataFields, max)
          return
        }
        if (+data <= min) {
          this.$set(this.dataFrom, dataFields, min)
          return
        }
        if (
          dataFields === 'price' ||
          dataFields === 'oriPrice' ||
          dataFields === 'volume' ||
          dataFields === 'weight' ||
          dataFields === 'skuScore'
        ) {
          this.$set(this.dataFrom, dataFields, this.checkInput(data))
        }
      }
      // if (dataFields === 'stocks') {
        this.$emit('input', this.lists)
      // }
    },

    /**
     * 只允许输入正数和小数(保留小数点后两位)
     */
    checkInput (num) {
      if (num) {
        var tmpVal = String(num).replace(/[^\d^\\.]/g, '')
        var reg = /^(0|([1-9]\d*))(\.\d{1,2})?$/ // 最多允许后输入两位小数
        if (!reg.test(tmpVal)) {
          tmpVal = tmpVal + ''
          tmpVal = tmpVal.substring(0, tmpVal.indexOf('.') + 3)
          var n = tmpVal.split('.').length - 1
          if (n > 1) {
            tmpVal = tmpVal.substring(0, tmpVal.indexOf('.'))
          }
        }
        return tmpVal
      } else {
        return ''
      }
    },

    /**
     * 编码输入框校验
     */
    validatePartyCode (scope) {
      const { row, $index } = scope
      // 纯空格校验
      if (validNoEmptySpace(row.partyCode)) {
        this.$set(this.lists[$index], 'partyCode', '')
        return
      }
      // 商品编码重复校验
      this.check(row, $index)
    },

    check (row, $index) {
      if (
        row.partyCode &&
        this.lists.find(
          (el, index) =>
            el.partyCode && index !== $index && el.partyCode === row.partyCode
        )
      ) {
        this.$message({
          message: this.$i18n.t('product.postProductTips16'),
          type: 'error',
          duration: 1500
        })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.mod-prod-sku-table {
  .pic-uploader-component .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    .pic-uploader-icon {
      font-size: 28px;
      color: #8c939d;
      width: 120px;
      height: 120px;
      line-height: 120px;
      text-align: center;
    }
    .pic {
      width: 120px;
      height: 120px;
      display: block;
    }
  }
  .pic-uploader-component .el-upload:hover {
    border-color: #409eff;
  }
  .del {
    color: #155bd4;
    cursor: pointer;
  }
  .tag-input-width.el-input-number--small {
    width: 100%;
  }
  .tag-input-width.party-code::placeholder {
    color: #999;
  }
  .tag-input-width.party-code.err-tips {
    border-color: #d40000;
  }

  // 表格输入框
  .tag-input-width {
    width: 100%;
    padding-left: 5px;
    padding-right: 0;
    border: 1px solid #dcdcdc;
    border-radius: 2px;
    height: 32px;
    line-height: 32px;
    box-sizing: border-box;
    &:focus {
      outline: 0;
    }
  }
  .tag-input-width.text-input {
    padding-right: 5px;
  }
  // 表格+批量设置
  .sku-table-con {
    display: block;
    padding: 10px;
    border: 1px solid #dcdcdc;
    .table-header {
      background: #f8f8f8;
    }

    // 批量设置
    .batch-settings-box {
      .set-txt {
        padding-top: 10px;
        .weak-txt {
          color: #999;
          font-size: 12px;
          margin-left: 5px;
        }
      }
      .batch-settings-tb {
        margin: 10px 0;
        // 头部
        ul,
        li {
          list-style: none;
          margin: 0;
          padding: 0;
          line-height: 1em;
        }
        .batch-settings-Head {
          display: flex;
          align-items: center;
          justify-content: space-between;
          padding: 15px 0;
          background: #f8f8f8;
          .stress {
            font-size: 14px;
            color: #d40000;
            margin-right: 3px;
          }
          li.head-item {
            flex: 1;
            padding: 0 10px;
            font-weight: bold;
          }
          .coding {
            width: 220px;
          }
        }

        .batch-settings-con {
          display: flex;
          align-items: center;
          justify-content: space-between;
          padding: 15px 0;
          .item {
            flex: 1;
            padding: 0 10px;
            & >>> .el-form-item,
            & >>> .el-form-item__content {
              width: 100%;
              margin-right: 0;
            }
          }
          .coding{
            width: 220px;
          }
        }
      }
    }
  }
}

// ::v-deep .el-table{
//   width: 76.5%;
// }

// div ::v-deep .el-form-item__content span,
// div ::v-deep .mul-pic-upload div {
//   color: #02A1E9;
// }
.filter-submitBtn span {
  color: #fff !important;
}
div >>> .el-table tbody tr:hover > td {
  background-color: #ffffff;
}
</style>
