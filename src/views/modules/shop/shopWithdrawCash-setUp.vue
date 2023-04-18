<template>
  <div>
    <el-dialog title="提现设置" :visible.sync="dialogFormVisible">
      <el-form @submit.native.prevent :inline="true"  class="demo-form-inline" :model="dataForm"  ref="dataForm">
        <el-form-item label="单笔提现"  >
          <el-input-number :max="999999" :precision="2"  v-model.trim="dataForm.withdrawCashLeast"  ><i slot="suffix" style="font-style:normal;margin-right: 10px;">元</i></el-input-number>
        </el-form-item>
        <el-form-item label="不高于" >
          <el-input @blur="
                    handleMaxValue(
                      dataForm.withdrawCashMax,
                      999999
                    )" v-model.trim="dataForm.withdrawCashMax"><i slot="suffix" style="font-style:normal;margin-right: 10px;">元</i></el-input>
        </el-form-item>
        <el-form-item label="提现频次"  >
        <el-select v-model="dataForm.frequency" >
          <el-option label="每周" :value=1></el-option>
          <el-option label="每月" :value=2></el-option>
          <el-option label="每年" :value=3></el-option>
        </el-select>
        <el-input-number v-model.trim="dataForm.number"  :min=1 :max="20"
                         @blur="handleEdit(dataForm.number)"
                         style="width: 140px;margin-left: 10px"><i slot="suffix" style="font-style:normal;margin-right: 10px;">次</i></el-input-number>
      </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
      <div class="default-btn" @click="dialogFormVisible = false">取消</div>
      <div class="default-btn primary-btn" @click="dataFormSubmit()">确认</div>
    </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      dialogFormVisible: false,
      dataForm: {
        withdrawCashLeast: '',
        withdrawCashMax: '',
        frequency: '',
        number: ''
      }
    }
  },
  methods: {
    open() {
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        this.$http({
          url: this.$http.adornUrl('/shop/shopWithdrawCash/getWithdrawCash'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.dataForm = data
        })
      })
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl('/shop/shopWithdrawCash/save'),
            method: 'post',
            data: this.$http.adornData(this.dataForm)
          }).then(({data}) => {
            this.$message({
              message: this.$t('shopWithdrawCash.success'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.dialogFormVisible = false
              }
            })
          })
        }
      })
    },
    handleMaxValue(data,max) {
      var patrn = /^(-)?\d+(\.\d+)?$/;
      if(!data.match(patrn)){
        this.dataForm.withdrawCashMax=''
        return;
      }
      if (data !== undefined && data !== null) {
        // 表格
        if (data > max) {
          this.dataForm.withdrawCashMax=max
          return
        }
      }
    },

    handleEdit(e) {
      this.dataForm.number = parseInt(e)
    }

  }
}
</script>

