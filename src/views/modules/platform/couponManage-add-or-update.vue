<template>
  <el-dialog
    class="mod-coupon-add-or-update"
    :title="dataForm.couponId ? $t('coupon.modifyCoupon') : $t('coupon.newCoupon')"
    :close-on-click-modal="false"
    :before-close="beforeClose"
    :visible.sync="visible"
  >
    <el-form @submit.native.prevent
      size="small"
      v-if="show"
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      :label-width="this.$i18n.t('language') === 'language' ? '170px' : '100px'"
    >
      <el-form-item :label="$t('coupon.couponName')" prop="couponName">
        <el-input
          v-model.trim="dataForm.couponName"
          :placeholder="$t('coupon.couponName')"
          maxlength="20"
          show-word-limit
          :disabled="dataForm.couponId !== 0"
        ></el-input>
      </el-form-item>
      <!-- <el-form-item :label="$t('coupon.couponSubtitle')" prop="subTitle">
        <el-input
          v-model.trim="dataForm.subTitle"
          :placeholder="$t('coupon.couponSubtitle')"
          maxlength="32"
          show-word-limit
          :disabled="dataForm.couponId !== 0"
        ></el-input>
      </el-form-item> -->
      <el-form-item :label="$t('coupon.deliveryStatus')" size="mini" prop="putonStatus">
        <el-radio-group v-model="dataForm.putonStatus" :disabled="dataForm.putonStatus < 0">
          <el-radio :label="0">{{$t("coupon.waitAutoLaunch")}}
            <el-tooltip class="item" effect="light" placement="top">
              <div slot="content">{{ $t("coupon.launchTip")}}</div>
              <span>
             <i class="el-icon-question"></i>
            </span>
            </el-tooltip>
          </el-radio>
          <el-radio :label="1">{{$t("coupon.launched")}}</el-radio>
          <el-radio :label="4">{{$t("coupon.waitLaunch")}}</el-radio>
          <el-radio :disabled="true" :label="-1">{{ $t("coupon.cancelLaunch") }}
            <el-tooltip class="item" effect="light" placement="top">
              <div slot="content">{{ $t("coupon.launchTip1")}}</div>
              <span>
             <i class="el-icon-question"></i>
            </span>
            </el-tooltip>
          </el-radio>
        </el-radio-group>
      </el-form-item>
      <!-- 投放时间 -->
      <el-form-item
        :label="this.$i18n.t('coupon.timeToMarket')"
        prop="launchTime"
        v-if="dataForm.putonStatus === 0"
      >
        <el-date-picker
          v-model="dataForm.launchTime"
          value-format="yyyy-MM-dd"
          type="date"
          style="width:140px"
          :placeholder="this.$i18n.t('coupon.chooseLaunchTime')"
        ></el-date-picker>
        <el-time-select
          v-model="launchTimeValue"
          :picker-options="{
            start: '00:00',
            step: '00:30',
            end: '23:30'
          }"
          size="small"
          style="width:100px"
          :placeholder="this.$i18n.t('coupon.startTime')">
        </el-time-select>
      </el-form-item>
      <el-form-item :label="$t('coupon.getWay')" size="mini" prop="getWay">
        <el-radio-group v-model="dataForm.getWay">
          <el-radio :label="0">{{$t("coupon.receiveDirectly")}}</el-radio>
          <el-radio :label="1">{{$t("coupon.exchangeOrSystemIssue")}}
            <el-tooltip class="item" effect="light" placement="top">
              <div slot="content">{{ $t("coupon.getWayTip")}}</div>
              <span>
              <i class="el-icon-question"></i>
            </span>
            </el-tooltip>
          </el-radio>
          <!-- <el-radio :label="3">兑换券</el-radio> -->
        </el-radio-group>
      </el-form-item>
      <el-form-item :label="$t('coupon.couponType')" size="mini" prop="couponType">
        <el-radio-group v-model="dataForm.couponType" :disabled="dataForm.couponId !== 0">
          <el-radio :label="1">{{$t("coupon.cashCoupon")}}</el-radio>
          <el-radio :label="2">{{$t("coupon.discountVoucher")}}</el-radio>
          <!-- <el-radio :label="3">兑换券</el-radio> -->
        </el-radio-group>
      </el-form-item>
      <el-form-item :label="$t('coupon.conditionsOfUse')" prop="cashCondition">
        {{$t('coupon.spendMoreThan')}}
        <el-input
          v-model="dataForm.cashCondition"
          :placeholder="$t('coupon.conditionsOfUse')"
          type="number"
          @change="cashConditionChange"
          min="0"
          :disabled="dataForm.couponId !== 0"
        >
          <template slot="append">{{$t('coupon.yuan')}}</template>
        </el-input>
      </el-form-item>
      <el-form-item
        :label="$t('coupon.reductionAmount')"
        prop="reduceAmount"
        v-if="dataForm.couponType === 1"
      >
        <el-input
          v-model="dataForm.reduceAmount"
          :placeholder="$t('coupon.reductionAmount')"
          type="number"
          @change="checkNumber(1)"
          :disabled="dataForm.couponId !== 0"
          :min="0.01"
          :max="99999"
        >
          <template slot="append">{{$t("coupon.yuan")}}</template>
        </el-input>
      </el-form-item>
      <el-form-item
        :label="$t('coupon.discountAmount')"
        prop="couponDiscount"
        v-if="dataForm.couponType === 2"
      >
        <el-input
          v-model="dataForm.couponDiscount"
          :placeholder="$t('coupon.discountAmount')"
          type="number"
          :disabled="dataForm.couponId !== 0"
          @change="checkNumber(2)"
          min="0"
        >
          <template slot="append">{{$t("coupon.off")}}</template>
        </el-input>
      </el-form-item>
      <!-- 生效时间 -->
      <el-form-item :label="$t('coupon.effectiveType')" np size="mini" prop="validTimeType">
        <el-radio-group v-model="dataForm.validTimeType" :disabled="dataForm.couponId !== 0">
          <el-radio :label="1">{{$t("coupon.fixedTime")}}</el-radio>
          <el-radio :label="2">{{$t("coupon.effectiveAfterReceiving")}}</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item
        :label="$t('coupon.startTime')"
        prop="startTime"
        v-if="dataForm.validTimeType === 1"
      >
        <el-date-picker
          :disabled="dataForm.couponId !== 0"
          v-model="dataForm.startTime"
          value-format="yyyy-MM-dd"
          type="date"
          size="small"
          style="width:140px"
          :placeholder="this.$i18n.t('marketing.chooseStartTime')"
        ></el-date-picker>
        <el-time-select
          v-model="startTimeValue"
          :picker-options="{
            start: '00:00',
            step: '00:30',
            end: '23:30'
          }"
          size="small"
          style="width:100px"
          :disabled="dataForm.couponId !== 0"
          :placeholder="this.$i18n.t('coupon.startTime')">
        </el-time-select>
        </el-form-item>
        <el-form-item
          :label="$t('coupon.endTime')"
          prop="endTime"
          v-if="dataForm.validTimeType === 1"
        >
        <el-date-picker
          v-model="dataForm.endTime"
          value-format="yyyy-MM-dd"
          type="date"
          style="width:140px"
          :placeholder="this.$i18n.t('marketing.chooseEndTime')"
        ></el-date-picker>
        <el-time-select
          v-model="endTimeValue"
          :picker-options="{
            start: '00:00',
            step: '00:30',
            end: '23:30'
          }"
          size="small"
          style="width:100px"
          :placeholder="this.$i18n.t('coupon.endTime')">
        </el-time-select>
      </el-form-item>
      <el-form-item
        :label="$t('coupon.AfterReceipt')"
        prop="afterReceiveDays"
        v-if="dataForm.validTimeType === 2"
      >
        <el-input
          v-model="dataForm.afterReceiveDays"
          type="number"
          style="width: 300px"
          oninput="if(value>3652)value=1"
          :max="3652"
          :disabled="dataForm.couponId !== 0"
        >
          <template slot="append">{{$t("marketing.effectiveDays")}}</template>
        </el-input>
        <el-tooltip class="item" effect="dark" :content="$t('marketing.maxTimeTip')" placement="top">
          <i class="el-icon-info"></i>
        </el-tooltip>
      </el-form-item>
      <el-form-item
        :label="$t('coupon.validDate')"
        prop="validDays"
        v-if="dataForm.validTimeType === 2"
      >
        <el-input
          v-model.number="dataForm.validDays"
          type="number"
          style="width: 300px"
          oninput="if(value>3652)value=1"
          :max="3652"
          :disabled="dataForm.couponId !== 0">
          <template slot="append">{{$t("coupon.day")}}</template>
        </el-input>
        <el-tooltip class="item" effect="dark" :content="$t('marketing.maxTimeTip')" placement="top">
          <i class="el-icon-info"></i>
        </el-tooltip>
      </el-form-item>
      <el-form-item :label="$t('coupon.restrictionsPerPerson')" prop="limitNum">
        <el-input v-model="dataForm.limitNum" type="number" min="1"  max="1000000000" @change="limitNumCheck">
          <template slot="append">{{$t("coupon.sheet")}}</template>
        </el-input>
      </el-form-item>
      <el-form-item :label="$t('coupon.stock')" prop="stocks">
        <el-input v-model="dataForm.stocks" type="number" min="1" max="1000000000" @change="stocksCheck" :disabled="dataForm.couponId !== 0">
          <template slot="append">{{$t("coupon.sheet")}}</template>
        </el-input>
      </el-form-item>
      <el-form-item :label="$t('coupon.applicableGoods')" size="mini" prop="suitableProdType">
        <el-radio-group v-model="dataForm.suitableProdType">
          <el-radio :label="0">{{$t("coupon.allProductsParticipate")}}</el-radio>
          <el-radio :label="1">{{$t("coupon.participateInDesignatedProd")}}</el-radio>
          <el-radio :label="2">{{$t("coupon.specifiedProductsDoNotParticipate")}}</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item>
        <div
          class="default-btn"
          v-if="dataForm.suitableProdType === 1 || dataForm.suitableProdType === 2"
          @click="prodsSelectHandle()"
        >{{$t("coupon.selectGoods")}}</div>
      </el-form-item>
      <!-- 商品 -->
      <el-form-item style="width:100%" v-if="dataForm.suitableProdType !== 0">
        <el-row>
          <div></div>
          <el-col v-for="(couponProd, index) in dataForm.couponProds" :key="index">
            <el-card :body-style="{ padding: '0px' }" style="height: 160px;width: 120px">
              <prod-pic
                  height="104px"
                  width="100%"
                  :pic="couponProd.pic"
                ></prod-pic>
              <div class="card-prod-bottom prod-line-height">
                <span class="card-prod-name">{{couponProd.prodName}}</span>
                <div
                  class="default-btn text-btn prod-text-left"
                  @click="deleteProd(index)"
                >{{$t("coupon.delete")}}</div>
              </div>
            </el-card>
          </el-col>
        </el-row>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <div class="default-btn" @click="show = false;visible = false">{{$t("coupon.cancel")}}</div>
      <div class="default-btn primary-btn" @click="dataFormSubmit()">{{$t("coupon.confirm")}}</div>
    </span>
    <!-- 商品选择弹窗-->
    <prods-select
      v-if="prodsSelectVisible"
      ref="prodsSelect"
      @refreshSelectProds="selectCouponProds"
    ></prods-select>
  </el-dialog>
</template>

<script>
import ProdsSelect from '@/components/prods-reserve-selection'
import ProdPic from '@/components/prod-pic'
import { getDateTimeRange, getParseTime } from '@/utils/datetime'
export default {
  data () {
    var validate = (rule, value, callback) => {
      // if (!/^[1-9]\d*$/.test(value) && !this.dataForm.putonStatus) {
      if (!/^[1-9]\d*$|^[1-9]\d*\.\d\d?$|^0\.\d\d?$/.test(Number(value))) {
        callback(new Error(this.$i18n.t('coupon.pleaseThan0')))
      } else {
        callback()
      }
    }
    var stockValidate = (rule, value, callback) => {
      // if (!/^[1-9]\d*$/.test(value) && !this.dataForm.putonStatus) {
      if (!/^[1-9]\d*$|^[1-9]\d*\.\d\d?$|^0\.\d\d?$/.test(value) && this.dataForm.couponId === 0) {
        callback(new Error(this.$i18n.t('coupon.pleaseThan0')))
      } else {
        callback()
      }
    }
    var discountValidate = (rule, value, callback) => {
      if (!/^[1-9]\d*$|^[1-9]\d*\.\d\d?$|^0\.\d\d?$/.test(Number(value))) {
        callback(new Error(this.$i18n.t('coupon.discountValidate')))
      } else if (parseInt(value) >= 10 || parseInt(value) < 0) {
        callback(new Error(this.$i18n.t('coupon.discountValidate')))
      } else {
        callback()
      }
    }
    var validateTime = (rule, value, callback) => {
      if (rule.field === 'startTime' && (!this.startTimeValue || !this.dataForm.startTime)) {
        callback(new Error(this.$i18n.t('formData.startTimeCannotBeEmpty')))
      }
      if (rule.field === 'endTime' && (!this.endTimeValue || !this.dataForm.endTime)) {
        callback(new Error(this.$i18n.t('formData.endTimeCannotBeEmpty')))
      }
      let startTime = this.dataForm.startTime + ' ' + this.startTimeValue + ':00'
      let endTime = this.dataForm.endTime + ' ' + this.endTimeValue + ':00'
      if (rule.field === 'startTime' && Date.parse(startTime) >= Date.parse(endTime)) {
        callback(new Error(this.$i18n.t('coupon.validateTime')))
      } if (rule.field === 'endTime' && new Date() > Date.parse(endTime)) {
        callback(new Error(this.$i18n.t('coupon.endTimeLimit')))
      } else {
        callback()
      }
    }
    return {
      show: true,
      visible: false,
      dataForm: {
        couponId: 0,
        couponName: '',
        subTitle: '',
        couponType: 1,
        reduceAmount: 0,
        couponDiscount: 0,
        cashCondition: 0,
        validTimeType: 1,
        getWay: 0,
        launchTime: null,
        startTime: null,
        endTime: null,
        afterReceiveDays: 0,
        validDays: 1,
        stocks: 1,
        suitableProdType: 0,
        limitNum: 1,
        putonStatus: 0,
        couponProds: []
      },
      isSubmit: false,
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      putonStatus: 0,
      errorTip: false,
      dataListSelections: [],
      prodsSelectVisible: false,
      resourcesUrl: process.env.VUE_APP_RESOURCES_URL,
      dataRule: {
        couponName: [
          { required: true, message: this.$i18n.t('coupon.couponNameCannotBeEmpty'), trigger: 'blur' }
        ],
        launchTime: [
          { required: true, message: this.$i18n.t('coupon.launchTimeTip1'), trigger: 'blur' }
        ],
        couponType: [
          { required: true, message: this.$i18n.t('coupon.couponTypeNotEmpty'), trigger: 'blur' }
        ],
        reduceAmount: [
          { required: true, message: this.$i18n.t('coupon.theDedEmpty'), trigger: 'blur' },
          { validator: validate, trigger: 'blur' }
        ],
        couponDiscount: [
          { required: true, message: this.$i18n.t('coupon.theDiscouBeEmpty'), trigger: 'blur' },
          { validator: discountValidate, trigger: 'blur' }
        ],
        startTime: [
          { required: true, message: this.$i18n.t('coupon.startTimeCannotBeEmpty'), trigger: 'blur' },
          { validator: validateTime, trigger: 'blur' }
        ],
        endTime: [
          { required: true, message: this.$i18n.t('coupon.endTimeCannotBeEmpty'), trigger: 'blur' },
          { validator: validateTime, trigger: 'blur' }
        ],
        afterReceiveDays: [
          { required: true, message: this.$i18n.t('coupon.timeCannotBeEmpty'), trigger: 'blur' }
        ],
        validDays: [
          { required: true, message: this.$i18n.t('coupon.timeCannotBeEmpty'), trigger: 'blur' },
          { type: 'number', min: 1, message: this.$i18n.t('coupon.validDaysAlert'), trigger: 'blur' }
        ],
        cashCondition: [
          { required: true, message: this.$i18n.t('coupon.conditionBeEmpty'), trigger: 'blur' },
          { validator: validate, trigger: 'blur' }
        ],
        validTimeType: [
          { required: true, message: this.$i18n.t('coupon.effectiveotBeEmpty'), trigger: 'blur' }
        ],
        stocks: [
          { required: true, message: this.$i18n.t('coupon.invenEmpty'), trigger: 'blur' },
          { validator: stockValidate, trigger: 'blur' }
        ],
        suitableProdType: [
          { required: true, message: this.$i18n.t('coupon.applicabltBeEmpty'), trigger: 'blur' }
        ]
      },
      launchTimeValue: '',
      startTimeValue: '',
      endTimeValue: ''
    }
  },
  components: {
    ProdsSelect,
    ProdPic
  },
  watch: {
    visible: function () {
      if (this.visible === false) {
        this.prodsSelectVisible = false
      }
    }
  },
  methods: {
    // 获取数据列表
    init (couponId) {
      this.show = true
      this.dataForm.couponId = couponId || 0
      this.putonStatus = 0
      this.visible = true
      this.isSubmit = false
      if (!couponId) { // 每次新增时  初始化表单数据
        this.initDataForm()
        let datetimeRange = getDateTimeRange()
        this.dataForm.launchTime = datetimeRange.startTime
        this.dataForm.startTime = datetimeRange.startTime
        this.dataForm.endTime = datetimeRange.endTime
        this.launchTimeValue = datetimeRange.currentTime
        this.startTimeValue = datetimeRange.currentTime
        this.endTimeValue = datetimeRange.currentTime
      }
      if (this.dataForm.couponId) {
        this.getDataList()
      }
    },
    // 初始化表单数据
    initDataForm () {
      this.dataForm = {
        couponId: 0,
        couponName: '',
        subTitle: '',
        couponType: 1,
        reduceAmount: 0,
        couponDiscount: 0,
        cashCondition: 0,
        validTimeType: 1,
        getWay: 0,
        launchTime: null,
        startTime: null,
        endTime: null,
        afterReceiveDays: 0,
        validDays: 1,
        stocks: 1,
        suitableProdType: 0,
        limitNum: 1,
        putonStatus: 0,
        couponProds: []
      }
    },
    getDataList () {
      this.$http({
        url: this.$http.adornUrl(`/platform/coupon/info/${this.dataForm.couponId}`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({ data }) => {
        this.dataForm = data
        this.launchTimeValue = this.dataForm.launchTime ? this.dataForm.launchTime.substring(11, this.dataForm.launchTime.length - 3) : ''
        this.startTimeValue = this.dataForm.startTime ? this.dataForm.startTime.substring(11, this.dataForm.startTime.length - 3) : ''
        this.endTimeValue = this.dataForm.endTime ? this.dataForm.endTime.substring(11, this.dataForm.endTime.length - 3) : ''
        this.dataForm.launchTime = getParseTime(this.dataForm.launchTime, '{y}-{m}-{d}')
        this.dataForm.startTime = getParseTime(this.dataForm.startTime, '{y}-{m}-{d}')
        this.dataForm.endTime = getParseTime(this.dataForm.endTime, '{y}-{m}-{d}')
        this.putonStatus = this.dataForm.putonStatus
      })
    },
    handleClose () {
      this.dataForm = {}
    },
    // 检验库存输入
    stocksCheck () {
      var maxNum = Math.round(this.dataForm.stocks)
      if (!maxNum) {
        maxNum = 1
      } else if (maxNum < 1) {
        maxNum = 1
      } else if (maxNum > 1000000000) {
        maxNum = 1000000000
      }
      this.dataForm.stocks = maxNum
    },
    // 检验限领张数输入
    limitNumCheck () {
      var maxNum = Math.round(this.dataForm.limitNum)
      if (!maxNum) {
        maxNum = 1
      } else if (maxNum < 1) {
        maxNum = 1
      } else if (maxNum > 1000000000) {
        maxNum = 1000000000
      }
      this.dataForm.limitNum = maxNum
    },
    // 选择点击事件
    selectChangeHandle (selection) {
      this.dataList.forEach((tableItem) => {
        let selectedProdIndex = selection.findIndex((selectedProd) => {
          if (!selectedProd) {
            return false
          }
          return selectedProd.prodId === tableItem.prodId
        })
        let dataSelectedProdIndex = this.dataListSelections.findIndex((dataSelectedProd) => dataSelectedProd.prodId === tableItem.prodId)
        if (selectedProdIndex > -1 && dataSelectedProdIndex === -1) {
          this.dataListSelections.push(tableItem)
        } else if (selectedProdIndex === -1 && dataSelectedProdIndex > -1) {
          this.dataListSelections.splice(dataSelectedProdIndex, 1)
        }
      })
    },
    // 显示添加指定商品弹框
    prodsSelectHandle () {
      this.prodsSelectVisible = true
      this.$nextTick(() => {
        this.$refs.prodsSelect.init(this.dataForm.couponProds)
      })
    },
    // 删除指定商品
    deleteProd (index) {
      this.dataForm.couponProds.splice(index, 1)
    },
    // 添加指定商品
    selectCouponProds (prods) {
      console.log(prods)
      if (prods) {
        this.dataForm.couponProds = prods
      }
    },
    /**
     * 输入框的数据改变时，对值进行校验
     */
    checkNumber (type) {
      // if(discountItem == null || discountItem.needAmount == null){

      // }
      // item.needAmount <= item.discount
      if (type === 1) {
        const reduceAmount = parseFloat(this.dataForm.reduceAmount)
        this.dataForm.reduceAmount = this.formatNumber(reduceAmount)
      }
      if (type === 2) {
        var coupDiscount = this.dataForm.couponDiscount
        // 如果小于零
        if (coupDiscount <= 0) {
          this.dataForm.couponDiscount = 0.1
        }
        if (coupDiscount >= 10) {
          // 如果折扣大于10
          this.dataForm.couponDiscount = 9.9
        }
      }
    },
    // 格式化输入的优惠券减免金额
    formatNumber (number) {
      number = parseFloat(number.toFixed(2))
      if (!number) {
        return
      }
      const maxNumber = 99999.99
      if (number >= maxNumber) {
        return maxNumber
      }
      if (number < 0.01) {
        return 0.01
      }
      return number
    },
    /**
     * 消费金额最低要求检验
     */
    cashConditionChange () {
      let caCondition = this.dataForm.cashCondition
      this.dataForm.cashCondition = caCondition <= 0 ? 0.01 : caCondition
      this.dataForm.cashCondition = caCondition > 9999999999999 ? 9999999999999 : caCondition
    },
    /**
     * 减免金额检验
     */
    reductionAmountChange () {
      let reduceAmount = this.dataForm.reduceAmount
      this.dataForm.reduceAmount = reduceAmount <= 0 ? 0.01 : reduceAmount
      this.dataForm.reduceAmount = reduceAmount > 9999999999999 ? 9999999999999 : reduceAmount
    },
    // 表单提交
    dataFormSubmit () {
      if (this.errorTip) {
        this.$message({
          message: this.$i18n.t('coupon.quantitssThan0'),
          type: 'error',
          duration: 1500
        })
        return
      }
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.dataForm.putonStatus === 0) {
            if (Date.parse(this.dataForm.launchTime + ' ' + this.launchTimeValue + ':00') <= Date.now()) {
              this.$message({
                message: this.$i18n.t('coupon.launchTimeTip'),
                type: 'error',
                duration: 1500
              })
              return
            } else if (this.dataForm.launchTime === null || this.launchTimeValue === null) {
              this.$message({
                message: this.$i18n.t('coupon.launchTimeTip1'),
                type: 'error',
                duration: 1500
              })
              return
            }
          }
          let launchTime = this.dataForm.launchTime
          let startTime = this.dataForm.startTime
          let endTime = this.dataForm.endTime
          this.dataForm.launchTime = this.dataForm.launchTime && this.launchTimeValue ? this.dataForm.launchTime + ' ' + this.launchTimeValue + ':00' : ''
          this.dataForm.startTime = this.dataForm.startTime && this.startTimeValue ? this.dataForm.startTime + ' ' + this.startTimeValue + ':00' : ''
          this.dataForm.endTime = this.dataForm.endTime && this.endTimeValue ? this.dataForm.endTime + ' ' + this.endTimeValue + ':00' : ''
          if (this.dataForm.couponType === 1 && (parseFloat(this.dataForm.cashCondition) <= parseFloat(this.dataForm.reduceAmount))) {
            this.$message.error(this.$i18n.t('coupon.amounnCannotBe'))
            return false
          }
          if (this.isSubmit) {
            return false
          }
          this.isSubmit = true
          this.$http({
            url: this.$http.adornUrl(`/platform/coupon`),
            method: this.dataForm.couponId ? 'put' : 'post',
            data: this.$http.adornData({
              'couponId': this.dataForm.couponId || undefined,
              'couponName': this.dataForm.couponName,
              'subTitle': this.dataForm.subTitle,
              'couponType': this.dataForm.couponType,
              'reduceAmount': parseFloat(this.dataForm.reduceAmount),
              'couponDiscount': parseFloat(this.dataForm.couponDiscount),
              'cashCondition': parseFloat(this.dataForm.cashCondition),
              'validTimeType': this.dataForm.validTimeType,
              'getWay': this.dataForm.getWay,
              'launchTime': this.dataForm.putonStatus === 0 ? this.dataForm.launchTime : null,
              'startTime': this.dataForm.startTime,
              'endTime': this.dataForm.endTime,
              'afterReceiveDays': this.dataForm.afterReceiveDays,
              'validDays': this.dataForm.validDays,
              'stocks': this.dataForm.stocks,
              'suitableProdType': this.dataForm.suitableProdType,
              'limitNum': this.dataForm.limitNum,
              'putonStatus': this.dataForm.putonStatus,
              'couponProds': this.dataForm.couponProds
            })
          }).then(({ data }) => {
            this.$message({
              message: this.$i18n.t('publics.operation'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.dataForm.couponProds = []
                this.$emit('refreshDataList', this.page)
                this.visible = false
              }
            })
          }).catch((e) => {
            this.isSubmit = false
          })
          this.dataForm.launchTime = launchTime
          this.dataForm.startTime = startTime
          this.dataForm.endTime = endTime
        }
      })
    },
    beforeClose (done) {
      this.show = false
      done()
    }
  }
}
</script>
<style lang="scss" scoped>
.mod-coupon-add-or-update {
  .el-col{
    width: 130px;
  }
}
.prod-line-height {
  line-height: 18px;
}
.prod-text-left {
  margin-left: 10px;
}
.el-date-editor >>>.el-input__suffix>.el-icon-circle-close {
  display: none
}
.el-date-editor >>>.el-input__suffix>.el-icon-circle-check {
  display: none
}
</style>
