<template>
  <div class="mod-hotSearch-add-or-update">
    <el-dialog title="修改等级" :close-on-click-modal="false" :visible.sync="visible" width="620px">
      <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm"
        @keyup.enter.native="dataFormSubmit()" label-width="80px">
        <el-form-item label="等级" prop="distributionInfoId">
          <el-select v-model="dataForm.distributionInfoId" size="small">
            <el-option v-for="item in dataList" :key="item.id" :label="item.name" :value="item.id"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <div class="default-btn" @click="visible = false">{{ $t('remindPop.cancel') }}</div>
        <div class="default-btn primary-btn" @click="openCheck()">{{ $t('remindPop.confirm') }}</div>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      dataForm: {
        distributionUserId: null,
        distributionInfoId: null
      },
      visible: false,
      dataRule: {
        distributionInfoId: [
          { required: true, message: '请选择等级', trigger: 'blur' }
        ]
      },
      dataList: []
    }
  },
  components: {
  },
  methods: {
    init(data) {
      console.log(data)
      this.visible = true
      if (!data.distributionInfo) {
        this.dataForm.distributionInfoId = null
      } else {
        this.dataForm.distributionInfoId = data.distributionInfo.id
      }
      this.dataForm.distributionUserId = data.distributionUserId
      this.getDistributionLevelList()
    },
    // 表单提交
    dataFormSubmit(row, index, done, loading) {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(`/distribution/distributionUser/update`),
            method: 'put',
            data: this.$http.adornData({
              distributionUserId: this.dataForm.distributionUserId,
              distributionInfoId: this.dataForm.distributionInfoId
            })
           
          }).then(({ data }) => {
            this.$message({
              message: this.$i18n.t('publics.operation'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.visible = false
                this.$emit('refreshDataList')
              }
            })
          })
        }
      })
    },
    openCheck() {
      this.dataFormSubmit()
    },
    // 获取会员等级列表
    getDistributionLevelList() {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/platform/distributionConfig/user/info'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({ data }) => {
        this.dataList = data
        this.dataListLoading = false
      })
    }
  }
}
</script>
