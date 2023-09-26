<template>
    <div>
        <el-dialog :title="!dataForm.id ? '新增' : '编辑'" :close-on-click-modal="false" :visible.sync="visible" width="600px">
            <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" size="small" ref="dataForm"
                @keyup.enter.native="dataFormSubmit()" label-width="80px">
                <el-form-item label="名称" prop="name">
                    <el-input v-model="dataForm.name" :disabled="dataForm.id"></el-input>
                </el-form-item>
                <el-form-item label="时间" prop="date">
                    <el-date-picker v-model="dataForm.date" type="datetime"
                        value-format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button type="primary" size="mini" @click="dataFormSubmit()">{{ $t('coupon.confirm') }}</el-button>
            </span>
        </el-dialog>

    </div>
</template>
  
<script>
import moment from 'moment'
export default {
    data() {
        return {
            visible: false,
            isSubmit: false,
            dataForm: {
                id: null,
                name: null,
                date: null,
            },
            dataRule: {
                name: [
                    { required: true, message: '名称不可为空', trigger: 'blur' }
                ],
                date: [
                    { required: true, message: '时间不可为空', trigger: 'blur' }
                ],
            }
        }
    },
    methods: {
        init(data) {
            this.dataForm.id = data?.id ? data?.id : 0 || 0
            this.visible = true
            this.$nextTick(() => {
                this.$refs['dataForm'].resetFields()
                this.dataForm.name = data.name
                this.dataForm.date = data.date
            })
        },

        // 表单提交
        dataFormSubmit() {
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    if (this.isSubmit) {
                        return false
                    }
                    this.$http({
                        url: this.$http.adornUrl(this.dataForm.id ? '/admin/user/update/resources' : '/admin/user/add/resources'),
                        method: 'post',
                        data: this.$http.adornData({
                            'id': this.dataForm.id ? this.dataForm.id : '',
                            'name': this.dataForm.name,
                            'date': this.dataForm.date
                        })
                    }).then(({ data }) => {
                        this.$message({
                            message: this.$i18n.t('remindPop.success'),
                            type: 'success',
                            duration: 1500,
                            onClose: () => {
                                this.visible = false
                                this.isSubmit = false
                                this.$emit('refreshDataList')
                            }
                        })
                    }).catch(() => {
                        this.isSubmit = false
                    })

                }
            })
        }
    }
}
</script>
  