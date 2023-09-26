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
            <div class="title">签到送豆设置</div>
            <el-form-item label="送多少豆">
                <el-input-number v-model="signInScoreOne" placeholder="观看时长(分钟)" controls-position="right" :min="0"
                    size="small" @change="scoreChange()"></el-input-number>
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
            score: {},
            signInScoreOne: 0,
            signInScoreTwo: 0,
            signInScoreThree: 0,
            signInScoreFour: 0,
            signInScoreFive: 0,
            signInScoreSix: 0,
            signInScoreSeven: 0,
            useDiscountRange: 0,
            getDiscountRange: 0,
            categoryConfigs: []
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

            // 送豆设置
            this.$http({
                url: this.$http.adornUrl('/user/scoreConfig/info/' + 'SCORE_CONFIG'),
                method: 'get',
                params: this.$http.adornParams()
            }).then(({ data }) => {
                if (data.signInScoreString) {
                    this.score = data
                    this.signInScoreOne = data.signInScore[0]
                    this.signInScoreTwo = data.signInScore[1]
                    this.signInScoreThree = data.signInScore[2]
                    this.signInScoreFour = data.signInScore[3]
                    this.signInScoreFive = data.signInScore[4]
                    this.signInScoreSix = data.signInScore[5]
                    this.signInScoreSeven = data.signInScore[6]
                    if (!data.useDiscountRange) {
                        this.useDiscountRange = 0
                    } else {
                        this.useDiscountRange = this.score.useDiscountRange
                    }
                    if (!data.getDiscountRange) {
                        this.getDiscountRange = 0
                    } else {
                        this.getDiscountRange = this.score.getDiscountRange
                    }
                }
                this.score.categoryConfigs = data.categoryConfigs
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
        },

        // 修改送豆设置
        scoreChange() {
            this.score.useDiscountRange = this.useDiscountRange
            this.score.getDiscountRange = this.getDiscountRange
            this.score.getDiscount = !this.score.getDiscount ? 0 : this.score.getDiscount
            this.score.useDiscount = !this.score.useDiscount ? 0 : this.score.useDiscount
            this.score.signInScore = []
            this.score.signInScore.push(this.signInScoreOne || 0)
            this.score.signInScore.push(this.signInScoreTwo || 0)
            this.score.signInScore.push(this.signInScoreThree || 0)
            this.score.signInScore.push(this.signInScoreFour || 0)
            this.score.signInScore.push(this.signInScoreFive || 0)
            this.score.signInScore.push(this.signInScoreSix || 0)
            this.score.signInScore.push(this.signInScoreSeven || 0)

            this.$http({
                url: this.$http.adornUrl('/user/scoreConfig'),
                method: 'post',
                data: this.$http.adornData(this.score)
            }).then(({ data }) => {
                this.$message({
                    message: this.$i18n.t('publics.operation'),
                    type: 'success',
                    duration: 1500,
                    onClose: () => {
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
  