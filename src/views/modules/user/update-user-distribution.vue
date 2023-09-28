<template>
  <div class="mod-hotSearch-add-or-update">
    <el-dialog title="绑定团长" :close-on-click-modal="false" :visible.sync="visible" width="620px">
      <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm"
        @keyup.enter.native="dataFormSubmit()" label-width="110px">
        <el-form-item label="用户信息" prop="realName">
          <div>
            <div>昵称：{{ dataForm.nickName }}</div>
            <div>手机号：{{ dataForm.userMobile }}</div>
          </div>
        </el-form-item>
        <el-form-item label="绑定团长" prop="cardNo">
          <el-select v-model="dataForm.cardNo" size="small" filterable clearable style="width:400px;">
            <el-option v-for="item in dataList" :key="item.cardNo"
              :label="item.realName + '-' + item.userMobile + '-' + item.station.stationName"
              :value="item.cardNo"></el-option>
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
        nickName: '', // 团长姓名
        distributionTel: '', // 自提门店
        cardNo: '', //邀请码
        userId: null, // 用户ID
      },
      visible: false,
      dataList: [],
      dataRule: {
        cardNo: [
          { required: true, message: '请选择团长', trigger: 'blur' },
        ],
      }
    }
  },
  components: {
  },
  methods: {
    init(data) {
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        this.dataForm.nickName = data.nickName
        this.dataForm.userMobile = data.userMobile
        this.dataForm.userId = data.userId
      })

      // 获取团长列表
      this.getDataList()
    },
    // 获取数据列表
    getDataList() {
      let theData = {
        current: 1,
        size: 1000000,
        sortParam: 1,
        sortType: 2
      }
      this.$http({
        url: this.$http.adornUrl('/distribution/distributionUser/page'),
        method: 'get',
        params: this.$http.adornParams(theData)
      }).then(({ data }) => {
        this.dataList = data.records
      })
    },
    openCheck() {
      this.dataFormSubmit()
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(`/admin/user/bindUser/${this.dataForm.cardNo}`),
            method: 'post',
            data: this.$http.adornData({
              userId: this.dataForm.userId,
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

  }
}
</script>
