<template>
    <div class="mod-api-userBalance">
        <div class="search-bar">
            <el-form @submit.native.prevent :inline="true" class="search-form" ref="searchForm" :model="searchForm"
                size="small">
                <div class="input-row">
                    <el-form-item prop="userMobile" label="团长手机号:">
                        <el-input v-model="searchForm.userMobile" type="text" clearable placeholder="手机号"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <div class="default-btn primary-btn" @click="searchChange(true)">{{ $t('crud.searchBtn') }}</div>
                        <div class="default-btn" @click="resetForm('searchForm')">{{ $t('product.reset') }}</div>
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
                    <el-table-column prop="userName" label="用户昵称">
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
                    <el-table-column label="日期">
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
export default {
    data() {
        return {
            theData: null, // 保存上次点击查询的请求条件
            theParams: null, // 保存上次点击查询的请求条件
            dataList: [],
            searchForm: {
                userMobile: null
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
            this.theParams = JSON.parse(JSON.stringify(this.searchForm))
            if (newData || !this.theData) {
                this.theData = {
                    current: page == null ? this.page.currentPage : page.currentPage,
                    size: page == null ? this.page.pageSize : page.pageSize,
                    userMobile: this.searchForm.userMobile
                }
            } else {
                this.theData.current = page == null ? this.page.currentPage : page.currentPage
                this.theData.size = page == null ? this.page.pageSize : page.pageSize
            }

            this.$http({
                url: this.$http.adornUrl('/distribution/distributionUser/get/list'),
                method: 'get',
                params: this.$http.adornParams(
                    Object.assign(this.theData, this.theParams), false
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
            this.searchForm.userMobile = null
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
  