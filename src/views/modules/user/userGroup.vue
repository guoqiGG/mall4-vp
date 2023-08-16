<template>
    <div class="mod-api-userBalance">
        <div class="search-bar">
            <el-form @submit.native.prevent :inline="true" class="search-form" ref="searchForm" :model="searchForm"
                size="small">
                <div class="input-row">
                    <el-form-item prop="groupName" label="分组名称:">
                        <el-input v-model="searchForm.groupName" type="text" clearable placeholder="名称"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <div class="default-btn primary-btn" @click="searchChange(true)">{{ $t('crud.searchBtn') }}</div>
                        <div class="default-btn" @click="resetForm('searchForm')">{{ $t('product.reset') }}</div>
                    </el-form-item>
                </div>
            </el-form>
        </div>
        <div class="main-container">
            <div class="operation-bar">
                <div v-if="isAuth('user:userGroup:save')" class="default-btn primary-btn"
                    @click="addOrUpdateHandle()">{{ $t('crud.addTitle') }}</div>
            </div>
            <div class="table-con">
                <el-table :data="dataList" header-cell-class-name="table-header" row-class-name="table-row"
                    style="width: 100%" ref="hotTable">
                    <el-table-column label="ID" prop="id">
                    </el-table-column>
                    <el-table-column prop="groupName" label="分组名称">
                    </el-table-column>
                    <el-table-column prop="userCount" label="分组数量" />
                    <el-table-column align="center" :label="$t('crud.menu')" width="220">
                        <template slot-scope="scope">
                            <div class="text-btn-con">
                                <div v-if="isAuth('user:userGroup:update')" class="default-btn text-btn btn-itm"
                                    @click="addOrUpdateHandle(scope.row.id)">{{ $t('crud.updateBtn') }}</div>
                                <div v-if="isAuth('user:userGroup:delete')" class="default-btn text-btn btn-itm"
                                    @click="deleteHandle(scope.row.id)">{{ $t('crud.delBtn') }}</div>
                            </div>
                        </template>
                    </el-table-column>
                </el-table>

            </div>
            <!-- 分页 -->
            <el-pagination v-if="dataList.length" @size-change="handleSizeChange" @current-change="handleCurrentChange"
                :current-page="page.currentPage" :page-sizes="[10, 20, 50, 100]" :page-size="page.pageSize"
                layout="total, sizes, prev, pager, next, jumper" :total="page.total">
            </el-pagination>
        </div>
        <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="refreshChange"></add-or-update>
    </div>
</template>
  
<script>
import AddOrUpdate from './userGroup-add-or-update'
export default {
    data() {
        return {
            theData: null, // 保存上次点击查询的请求条件
            theParams: null, // 保存上次点击查询的请求条件
            dataList: [],

            searchForm: {
                groupName: ''
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
        AddOrUpdate
    },
    created() {
    },
    mounted() {
        this.getDataList()
    },
    methods: {
        getDataList(page, newData = false) {
            this.dataListLoading = true
            if (newData || !this.theData) {
                this.theParams = JSON.parse(JSON.stringify(this.searchForm))
                this.theData = {
                    current: page == null ? this.page.currentPage : page.currentPage,
                    size: page == null ? this.page.pageSize : page.pageSize,
                    'groupName': this.searchForm.groupName
                }
            } else {
                this.theData.current = page == null ? this.page.currentPage : page.currentPage
                this.theData.size = page == null ? this.page.pageSize : page.pageSize
            }
            this.$http({
                url: this.$http.adornUrl('/admin/user/group/list'),
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
            this.searchForm.groupName = ''
        },
        // 新增 / 修改
        addOrUpdateHandle(id) {
            this.addOrUpdateVisible = true
            this.$nextTick(() => {
                this.$refs.addOrUpdate.init(id)
            })
        },
        deleteHandle(id) {
            this.$confirm(this.$t('seckill.isDeleOper'), this.$i18n.t('text.tips'), {
                confirmButtonText: this.$i18n.t('coupon.confirm'),
                cancelButtonText: this.$i18n.t('coupon.cancel'),
                type: 'warning'
            }).then(() => {
                this.$http({
                    url: this.$http.adornUrl('/admin/user/group/update'),
                    method: 'post',
                    data: this.$http.adornData({
                        id:id,
                        isDeleted:1
                    })
                }).then(({ data }) => {
                    this.$message({
                        message: this.$i18n.t('groups.successfulOperation'),
                        type: 'success',
                        duration: 1500,
                        onClose: () => {
                            this.refreshChange()
                        }
                    })
                })
            }).catch(() => { })
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
  