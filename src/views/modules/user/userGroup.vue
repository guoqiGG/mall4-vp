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
                <div v-if="isAuth('user:userGroup:save') && dataList.length < 10" class="default-btn primary-btn"
                    @click="addOrUpdateHandle()">{{ $t('crud.addTitle') }}</div>
            </div>
            <div class="table-con">
                <el-table :data="dataList" header-cell-class-name="table-header" row-class-name="table-row"
                    style="width: 100%" ref="hotTable">
                    <el-table-column label="ID" prop="groupId">
                    </el-table-column>
                    <el-table-column prop="groupName" label="分组名称">
                    </el-table-column>
                    <el-table-column prop="userCount" label="分组数量" />
                    <el-table-column align="center" :label="$t('crud.menu')" width="220">
                        <template slot-scope="scope">
                            <div class="text-btn-con">
                                <div v-if="isAuth('user:userGroup:update')" class="default-btn text-btn btn-itm"
                                    @click="addOrUpdateHandle(scope.row.groupId)">{{ $t('crud.updateBtn') }}</div>
                                <div v-if="isAuth('user:userGroup:delete')" class="default-btn text-btn btn-itm"
                                    @click="deleteHandle(scope.row.groupId)">{{ $t('crud.delBtn') }}</div>
                            </div>
                        </template>
                    </el-table-column>
                </el-table>
            </div>
        </div>
        <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="refreshChange"></add-or-update>
    </div>
</template>
  
<script>
import AddOrUpdate from './userGroup-add-or-update'
export default {
    data() {
        return {
            dataList: [],
            searchForm: {
                groupName: ''
            }, // 搜索
            dataListLoading: false,
            addOrUpdateVisible: false,
            resourcesUrl: process.env.VUE_APP_RESOURCES_URL
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
        getDataList() {
            this.dataListLoading = true
            this.$http({
                url: this.$http.adornUrl('/user/userGroup/list'),
                method: 'get',
                params: this.$http.adornParams(this.searchForm)
            }).then(({ data }) => {
                this.dataList = data
                this.dataListLoading = false
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
                    url: this.$http.adornUrl('/user/userGroup/' + id),
                    method: 'delete',
                    data: this.$http.adornData({})
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
  