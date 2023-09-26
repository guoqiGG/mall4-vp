<template>
    <div class="mod-api-userBalance">
        <div class="search-bar">
            <el-form @submit.native.prevent :inline="true" class="search-form" ref="searchForm" :model="searchForm"
                size="small">
                <div class="input-row">
                    <el-form-item label="日期:">
                        <el-date-picker v-model="searchForm.date" value-format="yyyy-MM-dd" type="date" placeholder="日期"
                            clearable>
                        </el-date-picker>
                    </el-form-item>

                    <el-form-item label="团长手机号">
                        <el-select v-model="searchForm.distributionId" filterable remote clearable placeholder="团长手机号"
                            :remote-method="distributionIdRemoteMethod" :loading="loading" class="select-parent">
                            <el-option v-for="item in distributionDataList" :key="item.distributionUserId"
                                :label="item.userMobile" :value="item.distributionUserId">
                            </el-option>
                        </el-select>
                    </el-form-item>

                    <!-- <el-form-item label="会员手机号">
                        <el-select v-model="searchForm.userId" filterable remote clearable placeholder="会员手机号"
                            :remote-method="userIdRemoteMethod" :loading="loading" class="select-parent">
                            <el-option v-for="item in userIdDataList" :key="item.userId" :label="item.userMobile"
                                :value="item.userId">
                            </el-option>
                        </el-select>
                    </el-form-item> -->

                    <el-form-item label="直播间名称">
                        <el-select v-model="searchForm.roomId" filterable remote clearable placeholder="直播间名称"
                            :remote-method="roomIdRemoteMethod" :loading="loading" class="select-parent">
                            <el-option v-for="item in roomIdDataList" :key="item.roomId" :label="item.name"
                                :value="item.roomId">
                            </el-option>
                        </el-select>
                    </el-form-item>


                    <el-form-item>
                        <div class="default-btn primary-btn" @click="searchChange(true)">{{ $t('crud.searchBtn') }}</div>
                        <div class="default-btn" @click="resetForm('searchForm')">{{ $t('product.reset') }}</div>
                        <div class="default-btn" @click="getSoldExcel()">导出</div>
                    </el-form-item>
                </div>
            </el-form>

            <div v-if="dataList !== null">
                观看时长：{{ dataList }}
            </div>
        </div>
    </div>
</template>
  
<script>
import moment from 'moment'

export default {
    data() {
        return {
            theData: null, // 保存上次点击查询的请求条件
            theParams: null, // 保存上次点击查询的请求条件
            dataList: null,
            distributionDataList: [],
            userIdDataList: [],
            roomIdDataList: [],
            searchForm: {
                date: '', //日期
                roomId: '',// 房间号
                // userId: '',// 用户ID
                distributionId: '',// 团长ID
            }, // 搜索
            dataListLoading: false,
            page: {
                total: 0, // 总页数
                currentPage: 1, // 当前页数
                pageSize: 10 // 每页显示多少条
            },
            loading: false
        }
    },
    components: {
    },
    created() {
    },
    mounted() {
    },
    methods: {
        getDataList(page, newData = false) {
            if (!this.searchForm.date) {
                this.$message({
                    message: '查询日期不能为空',
                    type: 'error',
                    duration: 1000
                })
                return
            }
            if (!this.searchForm.roomId) {
                this.$message({
                    message: '房间名称不能为空',
                    type: 'error',
                    duration: 1000
                })
                return
            }

            if (!this.searchForm.distributionId) {
                this.$message({
                    message: '团长手机号不能为空',
                    type: 'error',
                    duration: 1000
                })
                return
            }
            this.dataListLoading = true
            let params = {
            }

            if (this.searchForm.date) {
                params.date = this.searchForm.date
            }
            if (this.searchForm.roomId) {
                params.roomId = this.searchForm.roomId
            }
            // if (this.searchForm.userId) {
            //     params.userId = this.searchForm.userId
            // }
            if (this.searchForm.distributionId) {
                params.distributionId = this.searchForm.distributionId
            }

            this.theData = {
                current: page == null ? this.page.currentPage : page.currentPage,
                size: page == null ? this.page.pageSize : page.pageSize,
            }
            this.$http({
                url: this.$http.adornUrl('/platform/statistics/platform/room/date'),
                method: 'get',
                params: this.$http.adornParams(
                    Object.assign(this.theData, params), false
                )
            }).then(({ data }) => {
                this.dataList = data
            })
        },

        // 团长手机号模糊查询
        distributionIdRemoteMethod(value) {
            this.loading = true
            this.$http({
                url: this.$http.adornUrl('/distribution/distributionUser/page'),
                method: 'get',
                params: this.$http.adornParams(Object.assign({
                    current: 1,
                    size: 1000000,
                    userMobile: value
                }))
            }).then(({ data }) => {
                this.distributionDataList = data.records
                this.loading = false
            }).catch(() => {
                this.loading = false
            })
        },
        // 用户手机号模糊查询
        userIdRemoteMethod(value) {
            this.loading = true
            this.$http({
                url: this.$http.adornUrl('/admin/user/userPage'),
                method: 'get',
                params: this.$http.adornParams(Object.assign({
                    current: 1,
                    size: 1000000,
                    userMobile: value
                }))
            }).then(({ data }) => {
                this.userIdDataList = data.records
                this.loading = false
            }).catch(() => {
                this.loading = false
            })
        },
        // 房间号模糊查询
        roomIdRemoteMethod(value) {
            this.loading = true
            this.$http({
                url: this.$http.adornUrl('/live/liveRoom/page'),
                method: 'get',
                params: this.$http.adornParams(Object.assign({
                    current: 1,
                    size: 1000000,
                    name: value
                }))
            }).then(({ data }) => {
                this.roomIdDataList = data.records
                this.loading = false
            }).catch(() => {
                this.loading = false
            })
        },

        // 条件查询 JSON.stringify(arr)
        searchChange(newData = true) {
            this.getDataList(newData)
        },
        resetForm(formName) {
            this.$refs[formName].resetFields()
        },

        getSoldExcel() {
            if (!this.searchForm.date) {
                this.$message({
                    message: '查询日期不能为空',
                    type: 'error',
                    duration: 1000
                })
                return
            }
            if (!this.searchForm.roomId) {
                this.$message({
                    message: '房间名称不能为空',
                    type: 'error',
                    duration: 1000
                })
                return
            }

            if (!this.searchForm.distributionId) {
                this.$message({
                    message: '团长手机号不能为空',
                    type: 'error',
                    duration: 1000
                })
                return
            }

            let params = {
            }

            if (this.searchForm.date) {
                params.date = this.searchForm.date
            }
            if (this.searchForm.roomId) {
                params.roomId = this.searchForm.roomId
            }
            // if (this.searchForm.userId) {
            //     params.userId = this.searchForm.userId
            // }
            if (this.searchForm.distributionId) {
                params.distributionId = this.searchForm.distributionId
            }
            const loading = this.$loading({
                lock: true,
                target: '.mod-order-order',
                customClass: 'export-load',
                background: 'transparent',
                text: this.$i18n.t('shop.exportIng')
            })
            this.$http({
                url: this.$http.adornUrl('/platform/order/room/soldExcel'),
                method: 'get',
                params: this.$http.adornParams(params),
                responseType: 'blob' // 解决文件下载乱码问题
            }).then(({ data }) => {
                loading.close()
                var blob = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8' })
                const fileName = '观看时长表'
                const elink = document.createElement('a')
                if ('download' in elink) { // 非IE下载
                    elink.download = fileName
                    elink.style.display = 'none'
                    elink.href = URL.createObjectURL(blob)
                    document.body.appendChild(elink)
                    elink.click()
                    URL.revokeObjectURL(elink.href) // 释放URL 对象
                    document.body.removeChild(elink)
                } else { // IE10+下载
                    navigator.msSaveBlob(blob, fileName)
                }
            }).catch((e) => {
                loading.close()
            })
        }
    }
}
</script>
<style lang="scss" scoped>
.mod-api-userBalance {
    .main-container {
        margin: 0;
        padding: 0;
    }
}
</style>
  