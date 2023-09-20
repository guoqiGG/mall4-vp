<template>
    <div class="mod-coupon-add-or-update">
        <div class="new-page-title">
            <div class="line" />
            <div class="text">常规配置</div>
        </div>
        <el-form @submit.native.prevent ref="dataForm" @keyup.enter.native="dataFormSubmit()" class="form-box"
            label-width="auto" size="small">
            <div class="title">直播观看签到时长设置</div>
            <el-form-item label="观看时长(分钟)">
                <el-input-number v-model="viewTime.paramValue" placeholder="观看时长(分钟)" controls-position="right" :min="1"
                    size="small" @change="configChange(viewTime.id)"></el-input-number>
            </el-form-item>

            <div class="title">观看送豆设置</div>
            <el-form-item label="是否开启签到送豆">
                <el-radio-group v-model="open" @change="configChange(openViewTime.id)">
                    <el-radio :label="0">关闭</el-radio>
                    <el-radio :label="1">开启</el-radio>
                </el-radio-group>
            </el-form-item>

            <div class="title">观看多少分钟送礼物设置</div>
            <el-form-item label="观看时长(分钟)">
                <el-input-number v-model="gift.paramValue" placeholder="观看时长(分钟)" controls-position="right"
                    @change="configChange(gift.id)" :min="1" size="small"></el-input-number>
            </el-form-item>

        </el-form>
    </div>
</template>
<script>

export default {
    data() {
        return {
            viewTime: {}, // 113
            openViewTime: {},// 116
            open: null,
            gift: {}, //117
        }
    },
    created() {
        this.init()
    },
    methods: {
        // 获取数据列表
        init() {
            this.getDataList()
        },
        getDataList() {
            // 查询观看时长
            this.$http({
                url: this.$http.adornUrl(`/sys/config/info/113`),
                method: 'get',
            }).then(({ data }) => {
                this.viewTime = data
            })

            // 观看送豆设置
            this.$http({
                url: this.$http.adornUrl(`/sys/config/info/116`),
                method: 'get',
            }).then(({ data }) => {
                let aa = data
                let pattern = /\d+/g;
                let resultArray = aa.paramValue.match(pattern);
                let bb = { open: Number(resultArray[0]), deliver: Number(resultArray[1]) }
                aa.paramValue = bb
                this.open = bb.open
                this.openViewTime = aa
            })
            // 观看多少分钟送礼品
            this.$http({
                url: this.$http.adornUrl(`/sys/config/info/117`),
                method: 'get',
            }).then(({ data }) => {
                this.gift = data
            })
        },

        // 修改
        configChange(id) {
            console.log(this.openViewTime.paramValue.open)
            let data
            if (id == 113) {
                if (!this.viewTime.paramValue) {
                    this.viewTime.paramValue = 1
                }
                data = this.viewTime
            }
            if (id == 116) {
                this.openViewTime.paramValue.open = this.open
                let aa = this.openViewTime
                aa.paramValue = JSON.stringify(aa.paramValue)
                data = aa
            }
            if (id == 117) {
                if (!this.gift.paramValue) {
                    this.gift.paramValue = 1
                }
                data = this.gift
            }
            this.$http({
                url: this.$http.adornUrl(`/sys/config`),
                method: 'put',
                data: this.$http.adornData(data)
            }).then(({ data }) => {
                this.$message({
                    message: this.$i18n.t('publics.operation'),
                    type: 'success',
                    duration: 1500,
                    onClose: () => {
                        this.visible = false
                        this.$nextTick(() => {
                            this.init()
                        })
                    }
                })
            })
        }

    }
}
</script>
<style lang="scss" scoped>
.el-form-item {
    margin-top: 10px;
}

.title {
    font-weight: 600;
}
</style>
  