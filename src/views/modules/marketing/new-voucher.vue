<template>
  <div class="mod-coupon-add-or-update">
    <div class="new-page-title">
      <div class="line" />
      <div class="text">
        {{
          dataForm.voucherId==0
          ? '新增礼品券'
          : '编辑礼品券'
        }}
      </div>
    </div>
    <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm"
      @keyup.enter.native="dataFormSubmit()" class="form-box" label-width="auto">
      <el-form-item label="礼品券名称" prop="name">
        <el-input v-model="dataForm.name" maxlength="20" show-word-limit size="small" class="coupon-input"
          placeholder="礼品券名称"></el-input>
      </el-form-item>
      <!-- 固定时间数据 START -->
      <div>
        <el-form-item label="开始时间" prop="startTime">
          <el-date-picker :disabled="dataForm.voucherId !== 0" v-model="dataForm.startTime" value-format="yyyy-MM-dd"
            type="date" size="small" style="width:140px" placeholder="选择开始时间"></el-date-picker>
          <el-time-select v-model="startTimeValue" :picker-options="{
            start: '00:00',
            step: '00:30',
            end: '23:30'
          }" size="small" style="width:100px" :disabled="dataForm.voucherId !== 0"
            :placeholder="this.$i18n.t('coupon.startTime')">
          </el-time-select>
        </el-form-item>
        <el-form-item label="结束时间" prop="endTime">
          <el-date-picker size="small" v-model="dataForm.endTime" value-format="yyyy-MM-dd" type="date"
            style="width:140px" placeholder="选择结束时间"></el-date-picker>
          <el-time-select v-model="endTimeValue" :picker-options="{
            start: '00:00',
            step: '00:30',
            end: '23:30'
          }" size="small" style="width:100px" placeholder="结束时间">
          </el-time-select>
        </el-form-item>
      </div>
      <!-- 固定时间数据 END -->
      <!-- 每人限领 -->
      <el-form-item label="每人限领" prop="collar">
        <el-input v-model="dataForm.collar" size="small" @blur="toFloat('collar')" class="coupon-input1" type="number"
          style="width:240px">
          <template slot="append">{{ $t("marketing.piece") }}</template>
        </el-input>
      </el-form-item>
      <!-- 库存 -->
      <el-form-item label="库存" prop="number">
        <el-input v-model="dataForm.number" type="number" size="small" class="coupon-input1" @blur="toFloat('number')"
          style="width:240px" >
          <template slot="append">{{ $t("marketing.piece") }}</template>
        </el-input>
      </el-form-item>
      <el-form-item>
        <div class="default-btn" @click="back()">{{
          $t("crud.filter.cancelBtn")
        }}</div>
        <div class="default-btn primary-btn" @click="dataFormSubmit()">{{
          $t("crud.filter.submitBtn")
        }}</div>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { Debounce } from '@/utils/debounce'
import { getDateTimeRange, getParseTime } from '@/utils/datetime'
export default {
  data() {
    var validName = (rule, value, callback) => {
      if (!value.trim()) {
        callback(new Error('礼品券名称不能为空'))
      } else {
        callback()
      }
    }
    var validateCollar = (rule, value, callback) => {
      if (!/^[1-9]\d*$/.test(value)) {
        callback(new Error(this.$i18n.t('coupon.pleaseThan0')))
      } else if (value >= 2147483600) {
        callback(new Error(this.$i18n.t('coupon.stockLimit')))
      } else {
        callback()
      }
    }
    var validateNumber = (rule, value, callback) => {
      if (value === 0 && this.dataForm.voucherId) {
        callback()
      } else if (!/^[1-9]\d*$/.test(value)) {
        callback(new Error(this.$i18n.t('coupon.pleaseThan0')))
      } else if (value >= 2147483600) {
        callback(new Error(this.$i18n.t('coupon.stockLimit')))
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
        callback(new Error(this.$i18n.t('marketing.timeCanThanOrEqualTo')))
      }
      // if (this.dataForm.voucherId === 0 && rule.field === 'startTime' && new Date() > Date.parse(value)) {
      //   callback(new Error(this.$i18n.t('groups.activityTimeTime')))
      // }
      if (this.dataForm.voucherId === 0 && rule.field === 'endTime' && new Date() > Date.parse(endTime)) {
        callback(new Error(this.$i18n.t('groups.endTime')))
      } else {
        callback()
      }
    }
    return {
      dataForm: {
        voucherId: 0,
        name: '',
        startTime: null,
        endTime: null,
        number: 1,
        collar: 1,
        starUserId: null
      },
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      dataRule: {
        name: [
          { required: true, message: '礼品券名称不能为空', trigger: 'blur' },
          { validator: validName, trigger: 'blur' }
        ],
        collar: [
          { required: true, message: '限领数量不能为空', trigger: 'blur' },
          { validator: validateCollar, trigger: 'blur' }
        ],
        startTime: [
          { required: true, message: this.$i18n.t('formData.startTimeCannotBeEmpty'), trigger: 'blur' },
          { validator: validateTime, trigger: 'blur' }
        ],
        endTime: [
          { required: true, message: this.$i18n.t('formData.endTimeCannotBeEmpty'), trigger: 'blur' },
          { validator: validateTime, trigger: 'blur' }
        ],
        number: [
          { required: true, message: this.$i18n.t('marketing.invenEmpty'), trigger: 'blur' },
          { validator: validateNumber, trigger: 'blur' }
        ]
      },
      startTimeValue: '',
      endTimeValue: ''
    }
  },
  components: {
  },
  mounted() {
    const voucherId = this.$route.query.voucherId
    this.init(voucherId)
    let title = voucherId == 0 ? '新增礼品券' : '编辑礼品券'
    this.$store.commit('common/replaceSelectMenu', title)
  },
  methods: {
    // 获取数据列表
    init(voucherId) {
      this.dataForm.voucherId = voucherId || 0
      this.$nextTick(() => {
        this.$refs.dataForm.resetFields()
        let datetimeRange = getDateTimeRange()
        this.dataForm.startTime = datetimeRange.startTime
        this.dataForm.endTime = datetimeRange.endTime
        this.startTimeValue = datetimeRange.currentTime
        this.endTimeValue = datetimeRange.currentTime
        if (this.dataForm.voucherId) {
          this.getDataList()
        }
      })
    },
    getDataList() {
      this.$http({
        url: this.$http.adornUrl(`/platform/search/prod/get/one`),
        method: 'get',
        params: this.$http.adornParams({
          id: this.dataForm.voucherId
        })
      }).then(({ data }) => {
        this.dataForm = data
        this.dataForm.voucherId=data.id
        this.startTimeValue = this.dataForm.startTime ? this.dataForm.startTime.substring(11, this.dataForm.startTime.length - 3) : ''
        this.endTimeValue = this.dataForm.endTime ? this.dataForm.endTime.substring(11, this.dataForm.endTime.length - 3) : ''
        this.dataForm.startTime = getParseTime(this.dataForm.startTime, '{y}-{m}-{d}')
        this.dataForm.endTime = getParseTime(this.dataForm.endTime, '{y}-{m}-{d}')
      })
    },
    toFloat(type) {
      let val = parseFloat(this.dataForm[type])
      if (val) {
        this.dataForm[type] = val

        // 限制减免金额最多5位数字 99999.99
        if (type === 'reduceAmount') {
          this.dataForm[type] = this.formatNumber(val, 99999.99)
        }
        // 限制使用条件满元 9999999.99
        if (type === 'cashCondition') {
          this.dataForm[type] = this.formatNumber(val, 9999999.99)
        }
      }
    },
    // 表单提交
    dataFormSubmit: Debounce(function () {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          let startTime = this.dataForm.startTime
          let endTime = this.dataForm.endTime
          this.dataForm.startTime = this.dataForm.startTime && this.startTimeValue ? this.dataForm.startTime + ' ' + this.startTimeValue + ':00' : ''
          this.dataForm.endTime = this.dataForm.endTime && this.endTimeValue ? this.dataForm.endTime + ' ' + this.endTimeValue + ':00' : ''
          this.$http({
            url: this.$http.adornUrl(this.dataForm.voucherId == 0 ? '/platform/search/prod/add' : '/platform/search/prod/update'),
            method: 'post',
            data: this.$http.adornData({
              'id': this.dataForm.voucherId || undefined,
              'name': this.dataForm.name,
              'startTime': this.dataForm.startTime,
              'endTime': this.dataForm.endTime,
              'number': this.dataForm.number,
              'collar': this.dataForm.collar,
              'starUserId': this.dataForm.voucherId == 0 ? this.$store.state.user.id : this.dataForm.starUserId
            })
          }).then(({ data }) => {
            this.$message({
              message: this.$i18n.t('publics.operation'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.$emit('refreshDataList', this.page)
                this.back()
              }
            })
          })
          this.dataForm.startTime = startTime
          this.dataForm.endTime = endTime
        }
      })
    }, 1500),
    back() {
      this.$router.push('/marketing-voucher/voucher')
    }
  }
}
</script>
<style lang="scss" scoped>
.mod-coupon-add-or-update {
  .el-col {
    width: 130px;
  }

  .coupon-input {
    width: 520px;
  }

  .coupon-input1 {
    width: 220px;
  }

  .form-box {
    margin-left: 40px;
  }
}

.el-date-editor>>>.el-input__suffix>.el-icon-circle-close {
  display: none
}

.el-date-editor>>>.el-input__suffix>.el-icon-circle-check {
  display: none
}
</style>
