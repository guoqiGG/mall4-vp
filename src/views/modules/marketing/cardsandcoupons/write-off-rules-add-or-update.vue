<template>
  <el-dialog class="mod-coupon-add-or-update" :title="dataForm.id ? '编辑合成规则' : '新增合成规则'" :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm"
      @keyup.enter.native="dataFormSubmit()" label-width="150px" size="small">
      <el-form-item label="名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="名称" maxlength="21" show-word-limit></el-input>
      </el-form-item>

      <el-form-item label="批次号组" prop="stockIds">
        <el-input v-model="dataForm.stockIds" placeholder="批次号(在商家券表查询)" style="width:500px;">
        </el-input>
        <el-tooltip class="item" effect="light" content="批次号用逗号(,)拼接,最后末尾不需要逗号(示例:121313131313,1323131131)"
          placement="right" style="color: #155BD4;">
          <i class="el-icon-question" />
        </el-tooltip>
      </el-form-item>

      <el-form-item label="数量" prop="number">
        <el-input-number v-model="dataForm.number" controls-position="right" :min="1"
          placeholder="需要多少张数量来合成"></el-input-number>
      </el-form-item>
    </el-form>

    <span slot="footer" class="dialog-footer">
      <el-button @click="visible == false">{{ $t('coupon.cancel') }}</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>
<script>
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        id: null,
        name: '',
        stockIds: '',
        number: 1
      },
      dataRule: {
        name: [
          { required: true, message: '名称不能为空', trigger: 'blur' }
        ],
        stockIds: [
          { required: true, message: '批次号组不能为空', trigger: 'blur' }
        ],
        number: [
          { required: true, message: '数量不能为空', trigger: 'blur' }
        ]
      }
    }
  },
  components: {
  },
  watch: {

  },
  methods: {
    // 获取数据列表
    init(val) {
      this.dataForm.id = val.id || null
      this.visible = true
      this.$nextTick(() => {
        this.$refs.dataForm.resetFields()
      })
      this.$nextTick(() => {
        if (val.id) {
          this.dataForm.id = val.id
          this.dataForm.name = val.name
          this.dataForm.stockIds = val.stockIds
          this.dataForm.number = val.number
        }
      })
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.dataForm.id ? this.$http.adornUrl('/platform/shopCompany/update/synthesis') : this.$http.adornUrl('/platform/shopCompany/add/synthesis'),
            method: this.dataForm.id ? 'POST' : 'POST',
            data: this.$http.adornData(this.dataForm)
          }).then(({ data }) => {
            this.$message({
              message: this.dataForm.id ? '修改成功' : '创建成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.$emit('refreshDataList', this.page)
                this.visible = false
              }
            })
          })
        }
      })
    },
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
