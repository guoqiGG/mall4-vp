<template>
  <el-dialog class="mod-coupon-add-or-update" :title="dataForm.stock_id ? '编辑商家券' : '新增商家券'" :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm"
      @keyup.enter.native="dataFormSubmit()" label-width="150px" size="small">
      <el-form-item label="商家券批次名称" prop="stock_name">
        <el-input v-model="dataForm.stock_name" placeholder="商家券批次名称(示例值：8月1日活动券)" maxlength="21"
          show-word-limit></el-input>
      </el-form-item>
      <el-form-item label="适用商品范围" prop="goods_name">
        <el-input v-model="dataForm.goods_name" placeholder="适用商品范围(示例值：xxx商品使用)" maxlength="15"
          show-word-limit></el-input>
      </el-form-item>
      <el-form-item label="开始时间" prop="available_begin_time">
        <el-date-picker v-model="dataForm.coupon_use_rule.coupon_available_time.available_begin_time"
          value-format="yyyy-MM-dd HH:mm:ss" type="datetime" size="small" placeholder="开始时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="结束时间" prop="available_end_time">
        <el-date-picker v-model="dataForm.coupon_use_rule.coupon_available_time.available_end_time"
          value-format="yyyy-MM-dd HH:mm:ss" type="datetime" size="small" placeholder="结束时间"></el-date-picker>
      </el-form-item>
      <el-form-item label="小程序path" prop="mini_programs_path">
        <el-input v-model="dataForm.coupon_use_rule.mini_programs_path"
          placeholder="小程序path(示例值：/path/index/index)"></el-input>
      </el-form-item>
      <el-form-item label="批次最大发放个数" prop="max_coupons">
        <el-input v-model="dataForm.stock_send_rule.max_coupons" placeholder="批次最大发放个数" type="number" :max="1000000000"
          :min="5" @blur="handleMaxCoupons"></el-input>
      </el-form-item>
      <el-form-item label="用户最大可领个数" prop="max_coupons_per_user">
        <el-input v-model="dataForm.stock_send_rule.max_coupons_per_user" placeholder="用户最大可领个数" type="number" :max="100"
          :min="1" @blur="handleMaxCouponsPerUser"></el-input>
      </el-form-item>
    </el-form>

    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">{{ $t('coupon.cancel') }}</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>
<script>
import moment from 'moment'
export default {
  data() {

    var validateTime = (rule, value, callback) => {
      if (rule.field === 'available_begin_time' && (!this.dataForm.coupon_use_rule.coupon_available_time.available_begin_time)) {
        callback(new Error('开始时间不能为空'))
      } else if (rule.field === 'available_end_time' && (!this.dataForm.coupon_use_rule.coupon_available_time.available_end_time)) {
        callback(new Error('结束时间不能为空'))
      } else if (rule.field === 'available_begin_time' && Date.parse(this.dataForm.coupon_use_rule.coupon_available_time.available_begin_time) >= Date.parse(this.dataForm.coupon_use_rule.coupon_available_time.available_end_time)) {
        callback(new Error('开始时间不能大于等于结束时间'))
      } else if (rule.field === 'available_end_time' && Date.parse(this.dataForm.coupon_use_rule.coupon_available_time.available_begin_time) >= Date.parse(this.dataForm.coupon_use_rule.coupon_available_time.available_end_time)) {
        callback(new Error('结束时间不能小于等于结束时间'))
      } else {
        callback()
      }
    }


    var validatePath = (rule, value, callback) => {
      if (rule.field === 'mini_programs_path' && (!this.dataForm.coupon_use_rule.mini_programs_path)) {
        callback(new Error('小程序path不能为空'))
      } else {
        callback()
      }

    }
    var validateMaxCoupons = (rule, value, callback) => {
      if (rule.field === 'max_coupons' && (!this.dataForm.stock_send_rule.max_coupons)) {
        callback(new Error('批次最大发放个数不能为空'))
      } else {
        callback()
      }
    }
    var validateMaxCouponsPerUser = (rule, value, callback) => {
      if (rule.field === 'max_coupons_per_user' && (!this.dataForm.stock_send_rule.max_coupons_per_user)) {
        callback(new Error('用户最大可领个数不能为空'))
      } else {
        callback()
      }
    }

    return {
      visible: false,
      dataForm: {
        stock_id: null,
        stock_name: '',//商家券批次名称
        goods_name: '',//适用商品范围
        coupon_use_rule: { // 核销规则
          coupon_available_time: { // 券可核销时间
            available_begin_time: '', // 开始时间
            available_end_time: '',// 结束时间
          },
          exchange_coupon: {
            exchange_price: 0,
            transaction_minimum: 0,
          },
          mini_programs_path: '/package-activities/pages/my-card-package/my-card-package', //小程序path
        },
        stock_send_rule: { //券发放相关规则
          max_coupons: 5,//批次最大发放个数
          max_coupons_per_user: 1,//用户最大可领个数
        }
      },
      dataRule: {
        stock_name: [
          { required: true, message: '商家券批次名称不能为空', trigger: 'blur' }
        ],
        goods_name: [
          { required: true, message: '适用商品范围不能为空', trigger: 'blur' }
        ],
        available_begin_time: [
          { required: true, validator: validateTime, trigger: 'blur' },
        ],
        available_end_time: [
          { required: true, validator: validateTime, trigger: 'blur' },
        ],
        mini_programs_path: [
          { required: true, validator: validatePath, trigger: 'blur' }
        ],
        max_coupons: [
          { required: true, validator: validateMaxCoupons, trigger: 'blur' }
        ],
        max_coupons_per_user: [
          { required: true, validator: validateMaxCouponsPerUser, trigger: 'blur' }
        ],
      }
    }
  },
  components: {
  },
  watch: {

  },
  methods: {
    // 获取数据列表
    init(stock_id) {
      this.dataForm.stock_id = stock_id || null
      this.visible = true
      this.$nextTick(() => {
        this.$refs.dataForm.resetFields()
      })
      if (stock_id) {
        this.getDataList(stock_id)
      }
    },
    // 查询商家券详情API
    getDataList(stock_id) {
      this.$http({
        url: this.$http.adornUrl('/platform/shopCompany/get/wx/coupon/detail'),
        method: 'get',
        params: this.$http.adornParams(
          { stockId: stock_id }, false
        )
      }).then(({ data }) => {
        console.log(data)
        this.dataForm = data
      })
    },

    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          let data
          if (!this.dataForm.stock_id) {
            data = {
              stock_name: this.dataForm.stock_name,
              goods_name: this.dataForm.goods_name,
              coupon_use_rule: {
                coupon_available_time: {
                  available_begin_time: moment(this.dataForm.coupon_use_rule.coupon_available_time.available_begin_time).format(),
                  available_end_time: moment(this.dataForm.coupon_use_rule.coupon_available_time.available_end_time).format(),
                },
                exchange_coupon: {
                  exchange_price: 0,
                  transaction_minimum: 0,
                },
                mini_programs_path: '/package-activities/pages/my-card-package/my-card-package', //小程序path
              },
              stock_send_rule: {
                max_coupons: this.dataForm.stock_send_rule.max_coupons,
                max_coupons_per_user: this.dataForm.stock_send_rule.max_coupons_per_user
              }
            }
          } else {
            data = {
              stock_id: this.dataForm.stock_id,
              goods_name: this.dataForm.goods_name,
              coupon_use_rule: this.dataForm.coupon_use_rule
            }
          }

          this.$http({
            url: this.dataForm.stock_id ? this.$http.adornUrl(`https://api.mch.weixin.qq.com/v3/marketing/busifavor/stocks/${this.dataForm.stock_id}`) : this.$http.adornUrl('/platform/shopCompany/add/wx/coupon'),
            method: this.dataForm.stock_id ? 'Patch' : 'POST',
            data: this.$http.adornData(data)
          }).then(({ data }) => {
            this.$message({
              message: '创建成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.$emit('refreshDataList', this.page)
                this.visible = false
                this.dataForm.stock_send_rule.max_coupons=5
                this.dataForm.stock_send_rule.max_coupons_per_user=1
              }
            })
          })
        }
      })
    },
    handleMaxCoupons() {
      if (this.dataForm.stock_send_rule.max_coupons == 0 || !this.dataForm.stock_send_rule.max_coupons || this.dataForm.stock_send_rule.max_coupons == '' || !this.dataForm.stock_send_rule.max_coupons == null) {
        this.dataForm.stock_send_rule.max_coupons = 1
      }
      if (this.dataForm.stock_send_rule.max_coupons > 1000000000) {
        this.dataForm.stock_send_rule.max_coupons = 1000000000
      }
    },

    handleMaxCouponsPerUser() {
      if (this.dataForm.stock_send_rule.max_coupons_per_user == 0 || !this.dataForm.stock_send_rule.max_coupons_per_user || this.dataForm.stock_send_rule.max_coupons_per_user == '' || !this.dataForm.stock_send_rule.max_coupons_per_user == null) {
        this.dataForm.stock_send_rule.max_coupons_per_user = 1
      }
      if (this.dataForm.stock_send_rule.max_coupons_per_user > 100) {
        this.dataForm.stock_send_rule.max_coupons_per_user = 100
      }
    }
  }
}
</script>
<style lang="scss" scoped>
.mod-coupon-add-or-update {
  .prod-con {
    display: flex;
    flex-wrap: wrap;

    .el-card {
      margin: 0 15px 15px 0;
    }
  }
}

.use-rule {
  font-size: 16px;
  color: #000;
  font-weight: bold;
  margin-left: 30px;
}
</style>
