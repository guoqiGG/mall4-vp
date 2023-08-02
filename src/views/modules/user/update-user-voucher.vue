<template>
  <div>
    <el-dialog :title="$t('user.sendVouchers')" :visible.sync="visible" width="40%" class="select-coupon-dialog">
      <el-form @submit.native.prevent :inline="true" :model="searchForm" class="demo-form-inline form">
        <el-form-item label="当前数量:">
          <el-input-number controls-position="right" v-model="couponName" :min="0" size="small" :placeholder="$t('user.couponTip1')" clearable></el-input-number>
          <el-tooltip class="item" effect="light" placement="top">
            <div slot="content">输入多少，用户就有多少张礼品券</div>
            <span>
           <i class="el-icon-question"></i>
          </span>
          </el-tooltip>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <div @click="visible = false" class="default-btn">{{ $t('remindPop.cancel') }}</div>
        <div class="default-btn primary-btn" @click="submitProds()">{{ $t('remindPop.confirm') }}</div>
      </span>
    </el-dialog>
  </div>
</template>


<script>
import { Debounce } from '@/utils/debounce'
export default {

  data() {
    return {
      visible: false,
      confirmLoad: false,
      dataForm: {
        userIds: [],
        coupons: []
      },
      dataList: [], // 标签
      dataRule: {
      },
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      dataListLoading: false,
      searchForm: {},
      couponName: null,
      dataListSelections: []
    }
  },
  components: {
  },
  props: {
    value: {
      default: '',
      type: String
    },
    // 获取方式 null:默认（客户领取+平台发放） 0=客户领取 1=平台发放
    getWay: {
      default: null,
      type: Number
    },
    isSingle: {
      default: false,
      type: Boolean
    },
    // 最大上传数量
    limit: {
      default: 9,
      type: Number
    }
  },
  mounted() {
  },
  methods: {
    init(ids) {
      this.visible = true
      this.dataForm.userIds = ids
      this.getDataList()
    },
    // 每页数
    sizeChangeHandle(val) {
      this.page.pageSize = val
      this.page.currentPage = 1
      this.getDataList(this.page)
    },
    // 当前页
    currentChangeHandle(val) {
      this.page.currentPage = val
      this.getDataList(this.page)
    },
    // 分页获取标签
    getDataList(page) {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/platform/coupon/list'),
        method: 'get',
        params: this.$http.adornParams(
          Object.assign(
            {
              current: page == null ? this.page.currentPage : page.currentPage,
              size: page == null ? this.page.pageSize : page.pageSize,
              getWay: this.getWay
            },
            this.searchForm
          )
        )
      }).then(({ data }) => {
        this.dataList = data.records
        this.dataList.forEach(item => {
          // 平台投放 / 用户领取
          item.couponType = item.couponType === 1 ? this.$t('coupon.cashCoupon') : (item.couponType === 2 ? this.$t('coupon.discountVoucher') : (item.couponType === 3 ? this.$t('coupon.coinCertificate') : ''))
          // item.eachObtain = 1
          this.$set(item, 'eachObtain', 1)
        })
        this.page.total = data.total
        this.dataListLoading = false
      })
    },
    // 搜索
    searchChange() {
      this.searchForm.couponName = this.couponName
      this.getDataList(this.page)
    },
    // 单选商品事件
    getSelectProdRow(row) {
      // console.log('aa')
      this.dataListSelections = [row]
    },
    // 多选
    // 多选点击事件
    selectChangeHandle(selection) {
      this.dataList.forEach((tableItem) => {
        let selectedProdIndex = selection.findIndex((selectedCoupon) => {
          if (!selectedCoupon) {
            this.dataListSelections = []
            return false
          }
          return selectedCoupon.couponId === tableItem.couponId
        })
        let dataSelectedProdIndex = this.dataListSelections.findIndex((dataSelectedCoupon) => dataSelectedCoupon.couponId === tableItem.couponId)
        if (selectedProdIndex > -1 && dataSelectedProdIndex === -1) {
          this.dataListSelections.push(tableItem)
        } else if (selectedProdIndex === -1 && dataSelectedProdIndex > -1) {
          this.dataListSelections.splice(dataSelectedProdIndex, 1)
        }
      })
    },
    // 确定事件
    submitProds() {
      // 商品单选的情况
      // if (this.isSingle) {
      //   this.dataListSelections.length && this.$emit('refreshSelectCouponList', this.dataListSelections[0])
      //   this.dataListSelections = []
      //   this.visible = false
      //   return
      // }
      // // 多选
      // let coupons = []
      // // console.log('this.dataListSelections', this.dataListSelections, this.dataListSelections.length < 1)
      // if (this.dataListSelections.length < 1) {
      //   // this.$emit('refreshSelectCouponList', [])
      // } else {
      //   this.dataListSelections.forEach(item => {
      //     let couponIndex = coupons.findIndex((coupon) => coupon.couponId === item.couponId)
      //     if (couponIndex === -1) {
      //       coupons.push({ couponId: item.couponId, couponName: item.couponName, subTitle: item.subTitle, limitNum: item.limitNum, eachObtain: item.eachObtain })
      //     }
      //   })
      //   // this.$emit('refreshSelectCouponList', coupons)
      // }
      // this.dataForm.coupons = coupons
      // this.dataListSelections = []
      // this.confirm()
    },
    confirm: Debounce(function () {
      if (!this.dataForm.userIds) {
        return
      }
      if (!this.dataForm.coupons || !this.dataForm.coupons.length) {
        return this.$message.error(this.$t('coupon.pleaseSelectCoupon'))
      }
      this.confirmLoad = true
      let coupons = []
      this.dataForm.coupons.forEach(element => {
        let obj = {}
        obj.couponId = element.couponId
        obj.nums = element.eachObtain
        coupons.push(obj)
      })
      this.dataForm.coupons = coupons
      let param = this.dataForm
      console.log(param)
      this.$http({
        url: this.$http.adornUrl(`/platform/coupon/sendUserCoupon`),
        method: 'put',
        data: this.$http.adornData(param)
      }).then(({ data }) => {
        this.active = []
        this.$message({
          message: this.$i18n.t('remindPop.success'),
          type: 'success',
          duration: 1500,
          onClose: () => {
            this.visible = false
            this.confirmLoad = false
            this.$emit('refreshDataList', this.page)
          }
        })
      }).catch((e) => {
      })
    }, 1000),
    handleChange(row, index) {
      this.$set(this.dataList, index, this.dataList[index])
    }
  }
}
</script>
<style lang="scss" scoped>
.active {
  float: left;
  margin-left: 10px;
  padding: 10px;
  background: #e6a23c;
  margin-bottom: 10px;
  border-radius: 4px;
  font-size: 14px;
}

.Classification {
  float: left;
  margin-left: 10px;
  padding: 10px;
  background: #f7f7f7;
  margin-bottom: 10px;
  border-radius: 4px;
  font-size: 14px;
}

.select-coupon-dialog {
  .el-form-item {
    margin-bottom: 0;
  }

  .el-input.el-input--small {

  }

  .main-container {
    margin-bottom: 20px;
  }
}
</style>
