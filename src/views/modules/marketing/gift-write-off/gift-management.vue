<template>
    <div class="mod-marketing-coupon">
        <!-- 新版规范 -->
        <div class="coupon-mod">
            <!-- 表格主体 -->
            <div class="main-container">
                <div class="operation-bar">
                    <div class="default-btn primary-btn" v-if="isAuth('platform:giftManagement:save')"
                        @click="addOrUpdateHandle()">{{
                            $t("crud.addTitle") }}
                    </div>
                </div>
                <!-- 表格 -->
                <div class="table-con seckill-table">
                    <el-table :data="dataList" header-cell-class-name="table-header" row-class-name="table-row-low"
                        style="width: 100%">
                        <!-- <el-table-column fixed prop="id" label="ID" width="auto"></el-table-column> -->
                        <el-table-column prop="name" label="名称" width="auto"></el-table-column>
                        <el-table-column prop="date" label="日期" width="auto">
                            <template slot-scope="scope">
                                <span class="table-cell-text">{{ scope.row.date.split(' ')[0] }}</span>
                            </template>
                        </el-table-column>
                        <el-table-column prop="createTime" label="创建时间" width="auto"></el-table-column>
                        <el-table-column fixed="right" align="center" label="操作" width="180">
                            <template slot-scope="scope">
                                <div class="text-btn-con">
                                    <div class="default-btn text-btn" v-if="isAuth('platform:giftManagement:update')"
                                        @click="addOrUpdateHandle(scope.row.id)">修改</div>
                                    <!-- <div class="default-btn text-btn" @click="addOrUpdateHandle(scope.row.id)">下架</div> -->
                                    <div class="default-btn text-btn" v-if="isAuth('platform:giftManagement:update')"
                                        @click="deleteHandle(scope.row.id)">删除</div>
                                </div>
                            </template>
                        </el-table-column>
                    </el-table>
                </div>
            </div>
            <!-- 表格主体end -->
        </div>
        <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="refreshChange"></add-or-update>
    </div>
</template>
  
<script>
import moment from 'moment'
import addOrUpdate from './gift-management-add-or-update.vue'
export default {
    data() {
        return {
            dataList: [],
            dataListLoading: false,
            addOrUpdateVisible: false,
            // 头部搜索表单
            searchForm: {
                name: null
            }
        }
    },
    components: {
        addOrUpdate
    },
    mounted() {
        this.getDataList()
    },
    methods: {
        // 获取数据列表
        getDataList(page, newData = false) {
            this.dataListLoading = true
            this.$http({
                url: this.$http.adornUrl('/admin/user/get/resources/list'),
                method: 'get',
                params: this.$http.adornParams()
            }).then(({ data }) => {
                this.dataList = data
                this.dataListLoading = false
            }).catch(() => {
                this.dataListLoading = false
            })
        },
        // 新增 / 修改
        addOrUpdateHandle(id) {
            this.addOrUpdateVisible = true
            this.$nextTick(() => {
                this.$refs.addOrUpdate.init(id)
            })
        },
        // 删除
        deleteHandle(id) {
            this.$confirm(this.$t('seckill.isDeleOper'), this.$i18n.t('text.tips'), {
                confirmButtonText: this.$i18n.t('coupon.confirm'),
                cancelButtonText: this.$i18n.t('coupon.cancel'),
                type: 'warning'
            }).then(() => {
                this.$http({
                    url: this.$http.adornUrl('/admin/user/update/resources'),
                    method: 'post',
                    data: this.$http.adornData({
                        id: id,
                        isDeleted: 1
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
        // 刷新回调用
        refreshChange() {
            this.getDataList(this.page)
        },
    }
}
</script>
<style lang="scss" scoped>
.mod-order-order {
    .tit {
        display: flex;
        height: 45px;
        align-items: center;
    }

    .tit .item {
        padding: 0 10px;
        width: 10%;
        text-align: center;
    }

    .tit .product {
        width: 25%;
    }

    .prod-tit {
        padding: 10px;
        background: #f8f8f9;
        border-left: 1px solid #dddee1;
        border-top: 1px solid #dddee1;
        border-right: 1px solid #dddee1;
    }

    .prod-tit span {
        margin-right: 15px;
    }

    .prod-cont {
        display: flex;
        border-top: 1px solid #dddee1;
        border-bottom: 1px solid #dddee1;
        border-left: 1px solid #dddee1;
        color: #495060;
    }

    .prod-cont .item {
        display: flex;
        display: -webkit-flex;
        align-items: center;
        justify-content: center;
        padding: 10px;
        // width: 10%;
        border-right: 1px solid #dddee1;
        text-align: center;
        height: 100%;
    }

    .prod-cont .item span {
        display: block;
    }

    .prod-cont .prod-item {
        // width: 38%;
        display: flex;
        flex-direction: column;
        border-right: 1px solid #dddee1;
    }

    .prod-name {
        width: 55%;
        text-align: left;
    }

    .prod-price {
        position: absolute;
        right: 40px;
        text-align: center;
    }

    .prod-price span {
        display: block;
        margin-bottom: 10px;
    }

    .prod-name .prod-info {
        display: block;
        color: #80848f;
        margin-top: 30px;
    }

    .prod-cont .items.name {
        display: flex;
        position: relative;
        padding: 20px;
        // height: 100px;
        border-bottom: 1px solid #dddee1;
    }

    .prod-cont .items.name:last-child {
        border-bottom: none;
    }

    .prod-image {
        margin-right: 20px;
        width: 100px;
        height: 100px;
    }

    .prod-image img {
        width: 100px;
        height: 100px;
    }

    .item span {
        display: block;
        margin-bottom: 10px;
    }

    .item .operate {
        color: #2d8cf0;
    }

    .item .totalprice {
        color: #c00;
    }

    .prod .remark {
        width: 100%;
        height: 50px;
        line-height: 50px;
        background-color: #e8f7f6;
        border-left: 1px solid #dddee1;
        border-right: 1px solid #dddee1;
        border-bottom: 1px solid #dddee1;
        margin-bottom: 20px;
    }

    .buyer-remark {
        padding: 0 20px;
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
    }
}
</style>
  