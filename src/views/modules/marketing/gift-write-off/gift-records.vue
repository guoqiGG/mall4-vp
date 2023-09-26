<template>
    <div class="mod-api-userBalance">
        <div class="search-bar">
            <el-form @submit.native.prevent :inline="true" class="search-form" ref="searchForm" :model="searchForm"
                size="small">
                <div class="input-row">
                    <el-form-item prop="userMobile" label="团长手机号:">
                        <el-input v-model="searchForm.userMobile" type="text" clearable placeholder="手机号"></el-input>
                    </el-form-item>
                    <!-- <el-form-item prop="tel" label="会员手机号:">
                        <el-input v-model="searchForm.tel" type="text" clearable placeholder="会员手机号"></el-input>
                    </el-form-item> -->
                    <el-form-item prop="scoreName" label="礼物名称:">
                        <el-input v-model="searchForm.scoreName" type="text" clearable placeholder="礼物名称"></el-input>
                    </el-form-item>
                    <el-form-item label="日期:">
                        <el-date-picker size="small" v-model="date" type="daterange" :range-separator="$t('date.tip')"
                            value-format="yyyy-MM-dd HH:mm:ss" start-placeholder="开始时间" end-placeholder="结束时间"
                            @change="createTimeChange"></el-date-picker>
                    </el-form-item>

                    <el-form-item>
                        <div class="default-btn primary-btn" @click="searchChange(true)">{{ $t('crud.searchBtn') }}</div>
                        <div class="default-btn" @click="resetForm('searchForm')">{{ $t('product.reset') }}</div>
                        <div class="default-btn" @click="getSoldExcel()">导出</div>
                    </el-form-item>
                </div>
            </el-form>
        </div>
        <div class="main-container">
            <div class="table-con">
                <el-table :data="dataList" header-cell-class-name="table-header" row-class-name="table-row"
                    style="width: 100%" ref="hotTable">
                    <el-table-column prop="distributionName" label="团长">
                    </el-table-column>
                    <el-table-column prop="userModile" label="团长手机号">
                    </el-table-column>
                    <el-table-column prop="userName" label="团员">
                    </el-table-column>
                    <el-table-column label="直播间图片">
                        <template slot-scope="scope">
                            <el-image style="width: 75px; height: 75px" :src="scope.row.coverImg" fit="fill"></el-image>
                        </template>
                    </el-table-column>
                    <el-table-column prop="roomName" label="直播间标题">
                    </el-table-column>
                    <el-table-column prop="bizName" label="礼物名称">
                    </el-table-column>
                    <el-table-column label="核销日期">
                        <template slot-scope="scope">
                            {{ scope.row.date.split(' ')[0] }}
                        </template>
                    </el-table-column>
                    <!-- <el-table-column align="center" :label="$t('crud.menu')" width="220">
                        <template slot-scope="scope">
                            <div class="text-btn-con">
                                <div v-if="isAuth('platform:giftRecords:update')" class="default-btn text-btn btn-itm"
                                    @click="addOrUpdateHandle(scope.row.id)">{{ $t('crud.updateBtn') }}</div>
                                <div v-if="isAuth('platform:giftRecords:delete')" class="default-btn text-btn btn-itm"
                                    @click="deleteHandle(scope.row.id)">{{ $t('crud.delBtn') }}</div>
                            </div>
                        </template>
                    </el-table-column> -->
                </el-table>

            </div>
            <!-- 分页 -->
            <el-pagination v-if="dataList.length" @size-change="handleSizeChange" @current-change="handleCurrentChange"
                :current-page="page.currentPage" :page-sizes="[10, 20, 50, 100]" :page-size="page.pageSize"
                layout="total, sizes, prev, pager, next, jumper" :total="page.total">
            </el-pagination>
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
            dataList: [],
            date: '',//日期
            searchForm: {
                userMobile: null, // 团长手机号
                // tel: null, // 会员手机号
                scoreName: '',// 礼物名称
                starTime: '',//开始时间
                endTime: '',//结束时间
                type: 0
            }, // 搜索
            dataListLoading: false,
            addOrUpdateVisible: false,
            page: {
                total: 0, // 总页数
                currentPage: 1, // 当前页数
                pageSize: 10 // 每页显示多少条
            },
        }
    },
    components: {
    },
    created() {
    },
    mounted() {
        this.getDataList()
    },
    methods: {
        getDataList(page, newData = false) {
            this.dataListLoading = true
            let params = {
                type: 0
            }
            if (this.searchForm.userMobile) {
                params.userMobile = this.searchForm.userMobile
            }
            // if (this.searchForm.tel) {
            //     params.tel = this.searchForm.tel
            // }
            if (this.searchForm.scoreName) {
                params.scoreName = this.searchForm.scoreName
            }
            if (this.searchForm.starTime) {
                params.starTime = this.searchForm.starTime
            }
            if (this.searchForm.endTime) {
                params.endTime = this.searchForm.endTime
            }

            this.theData = {
                current: page == null ? this.page.currentPage : page.currentPage,
                size: page == null ? this.page.pageSize : page.pageSize,
            }

            this.$http({
                url: this.$http.adornUrl('/distribution/distributionUser/get/list'),
                method: 'get',
                params: this.$http.adornParams(
                    Object.assign(this.theData, params), false
                )
            }).then(({ data }) => {
                this.dataList = data.records
                this.page.total = data.total
                this.dataListLoading = false

                // 末尾页数据为空重新请求
                if (!this.dataList.length && this.page.currentPage > 1) {
                    this.page.currentPage = 1
                    this.getDataList()
                }
            })
        },
        // 条件查询 JSON.stringify(arr)
        searchChange(newData = true) {
            this.getDataList(newData)
        },
        resetForm(formName) {
            this.$refs[formName].resetFields()
            this.date = null
        },
        /**
         * 刷新回调
         */
        refreshChange() {
            this.getDataList(this.page)
        },
        searchChange() {
            this.getDataList(this.page)
        },
        // 每页数量变更
        handleSizeChange(val) {
            this.page.pageSize = val
            this.getDataList()
        },
        // 页数变更
        handleCurrentChange(val) {
            this.page.currentPage = val
            this.getDataList()
        },
        createTimeChange() {
            console.log(this.date)
            if (!this.date || this.date.length === 0) {
                this.searchForm.starTime = null
                this.searchForm.endTime = null
            } else {
                this.searchForm.starTime = this.date[0].split(' ')[0]
                this.searchForm.endTime = this.date[1].split(' ')[0]

            }
        },
        getSoldExcel() {
            let params = {
                type: 0
            }
            if (this.searchForm.userMobile) {
                params.userMobile = this.searchForm.userMobile
            }
            // if (this.searchForm.tel) {
            //     params.tel = this.searchForm.tel
            // }
            if (this.searchForm.scoreName) {
                params.scoreName = this.searchForm.scoreName
            }
            if (this.searchForm.starTime) {
                params.starTime = this.searchForm.starTime
            }
            if (this.searchForm.endTime) {
                params.endTime = this.searchForm.endTime
            }
            const loading = this.$loading({
                lock: true,
                target: '.mod-order-order',
                customClass: 'export-load',
                background: 'transparent',
                text: this.$i18n.t('shop.exportIng')
            })
            this.$http({
                url: this.$http.adornUrl('/platform/order/resources/excel'),
                method: 'get',
                params: this.$http.adornParams(params),
                responseType: 'blob' // 解决文件下载乱码问题
            }).then(({ data }) => {
                loading.close()
                var blob = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8' })
                const fileName = '核销礼物记录'
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
  