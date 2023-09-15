<template>
  <el-dialog class="mod-coupon-add-or-update" title="查看券详情" :close-on-click-modal="false" :visible.sync="visible"
    >
    <el-form @submit.native.prevent :model="dataForm" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
      label-width="150px" size="small" :disabled="true">
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
      <!-- UNAUDIT：审核中
            RUNNING：运行中
            STOPED：已停止 （暂未开放）
            PAUSED：暂停发放 （暂未开放） -->
      <el-form-item label="状态" prop="stock_state">
        <el-input v-model="dataForm.stock_state" 
        ></el-input>
      </el-form-item>

      <el-form-item label="批次最大发放个数" prop="max_coupons">
        <el-input v-model="dataForm.stock_send_rule.max_coupons" placeholder="批次最大发放个数" type="number" :max="1000000000" 
          :min="5"></el-input>
      </el-form-item>
      <el-form-item label="用户最大可领个数" prop="max_coupons_per_user">
        <el-input v-model="dataForm.stock_send_rule.max_coupons_per_user" placeholder="用户最大可领个数" type="number" :max="100" 
          :min="1"></el-input>
      </el-form-item>
      <el-form-item label="总发放张数" prop="total_send_num">
        <el-input v-model="dataForm.send_count_information.total_send_num" type="number" :max="100" 
          :min="1"></el-input>
      </el-form-item>

    </el-form>

    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">关闭</el-button>
    </span>
  </el-dialog>
</template>
<script>
import moment from 'moment'
const statusList = [
  { code: 'UNAUDIT', name: '审核中' },
  { code: 'RUNNING', name: '运行中' },
  { code: 'STOPED', name: '已停止 （暂未开放）' },
  { code: 'PAUSED', name: '暂停发放 （暂未开放）' }
]
export default {
  data() {
    return {
      visible: false,
      dataForm: {
      },
    }
  },
  methods: {
    // 获取数据列表
    init(stock_id) {
      this.dataForm.stock_id = stock_id
      this.visible = true
      this.getDataList(stock_id)
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
        statusList.forEach(e=>{
          if(e.code==this.dataForm.stock_state){
            this.dataForm.stock_state=e.name
          }
        })
      })
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
