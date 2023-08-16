<template>
  <el-dialog :title="$t('user.updateUserGroup')" :visible.sync="visible" width="30%">
    <el-form @submit.native.prevent :model="dataForm" ref="dataForm" :rules="dataRule"
      :label-width="this.$i18n.t('language') === 'language' ? '160px' : '100px'">
      <el-form-item label="分组名称" prop="groupId">
        <el-select size="small" v-model="dataForm.groupId" clearable placeholder="请选择分组" class="select-group-box-item">
          <el-option
            v-for="item in dataList"
            :key="item.id"
            :label="item.groupName"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <div class="default-btn" @click="visible = false">{{ $t('remindPop.cancel') }}</div>
        <div class="default-btn primary-btn" @click="confirm">{{ $t('remindPop.confirm') }}</div>
      </el-form-item>
    </el-form>
  </el-dialog>
</template>


<script>
import { Debounce } from '@/utils/debounce'
export default {

  data() {
    return {
      visible: false,
      confirmLoad: false,
      dataForm: {
        groupId: null,
        userIds: []
      },
      dataList:[],
      dataRule: {
        groupId: [
          { required: true, message: '请选择分组', trigger: 'blur' },
        ]
      }
    }
  },
  methods: {
    init(ids) {
      this.visible = true
      this.dataForm.userIds = ids
      this.dataForm.groupId = null
      this.getDataList()
    },

    // 获取会员分组列表
    getDataList() {
      this.$http({
        url: this.$http.adornUrl('/admin/user/group/list'),
        method: 'get',
        params: this.$http.adornParams(
          Object.assign({current:1,size:10000}), false
        )
      }).then(({ data }) => {
        this.dataList = data.records
      })
    },
    confirm: Debounce(function () {
      this.$refs['dataForm'].validate((valid) => {
        if (!valid) {
          return
        }
        if (!this.dataForm.userIds) {
          return
        }
        this.confirmLoad = true
        let param = this.dataForm
        this.$http({
          url: this.$http.adornUrl(`/admin/user/update/relationship`),
          method: 'post',
          data: this.$http.adornData(param)
        }).then(({ data }) => {
          this.$message({
            message: this.$i18n.t('remindPop.success'),
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.visible = false
              this.confirmLoad = false
              this.$emit('refreshDataList', this.page)
              this.dataForm.groupId = null
            }
          })
        }).catch((e) => {
        })
      })
    }, 1000)
  }
}
</script>
