<template>
  <div class="mod-hotSearch-add-or-update">
    <el-dialog title="更换团长" :close-on-click-modal="false" :visible.sync="visible" width="620px">
      <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm"
        @keyup.enter.native="dataFormSubmit()" label-width="80px">
        <el-form-item label="团长姓名" prop="realName">
          {{ dataForm.realName }}
        </el-form-item>
        <el-form-item label="新团长" prop="newId">
          <el-select v-model="dataForm.newId" size="small" filterable clearable>
            <el-option v-for="item in dataList" :key="item.distributionUserId" :label="item.realName"
              :value="item.distributionUserId"></el-option>
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
    var newIdValidator = (rule, value, callback) => {
      if (this.dataForm.id == value) {
        callback(new Error('更换的新团长和原团长不能相同'))
      } else {
        callback()
      }
    }
    return {
      dataForm: {
        realName: '',
        id: null, //原团长ID
        newId: null, // 新团长ID
      },
      visible: false,
      dataList: [],
      dataRule: {
        newId: [
          { required: true, message: '请选择新团长', trigger: 'blur' },
          { required: true, validator: newIdValidator, trigger: 'change' }
        ],
      }
    }
  },
  components: {
  },
  methods: {
    init(data) {
      console.log(data)
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        this.dataForm.realName = data.realName
        this.dataForm.id = data.distributionUserId
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
            url: this.$http.adornUrl(`/distribution/distributionUser/update/distribution/batch`),
            method: 'get',
            params: this.$http.adornParams({
              id: this.dataForm.id,
              newId: this.dataForm.newId
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
