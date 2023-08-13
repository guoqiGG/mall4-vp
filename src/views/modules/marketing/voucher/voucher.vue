<template>
  <div class="mod-marketing-coupon">

    <!-- 新版规范 -->
    <div class="coupon-mod">
      <!-- 搜索栏 -->
      <div class="search-bar">
        <el-form @submit.native.prevent :inline="true" class="search-form" ref="test-form" :model="searchForm"
          size="small">
          <div class="input-row">
            <el-form-item label="礼品券名:" class="search-form-item">
              <el-input v-model="searchForm.name" placeholder="礼品券名"></el-input>
            </el-form-item>
            <!-- 日期组件 -->
            <el-form-item :label="$t('coupon.expire') + ':'">
              <el-date-picker v-model="dateRange" type="datetimerange" :range-separator="$t('date.tip')"
                value-format="yyyy-MM-dd HH:mm:ss" :start-placeholder="$t('date.start')"
                :end-placeholder="$t('date.end')"></el-date-picker>
            </el-form-item>
            <el-form-item>
              <div class="default-btn primary-btn" @click="searchChange(true)">{{ $t('shopFeature.searchBar.search') }}
              </div>
              <div class="default-btn" @click="clearSearch">{{ $t('product.reset') }}</div>
            </el-form-item>
          </div>
        </el-form>
      </div>
      <!-- 搜索栏end -->
      <!-- 表格主体 -->
      <div class="main-container">
        <div class="operation-bar">
          <div class="default-btn primary-btn" v-if="isAuth('platform:voucher:save')" @click="addOrUpdateHandle()">{{
            $t("crud.addTitle") }}
          </div>
        </div>
        <!-- 表格 -->
        <div class="table-con seckill-table">
          <el-table :data="dataList" header-cell-class-name="table-header" row-class-name="table-row-low"
            style="width: 100%">
            <el-table-column fixed prop="name" label="礼品券名称" min-width="200">
              <template slot-scope="scope">
                <div>
                  <span class="table-cell-text">{{ scope.row.name }}</span>
                </div>
              </template>
            </el-table-column>

            <el-table-column prop="startTime" :label="$t('coupon.startTime')" min-width="160">
              <template slot-scope="scope">
                <span class="table-cell-text">{{ scope.row.startTime || '-' }}</span>
              </template>
            </el-table-column>

            <el-table-column prop="endTime" :label="$t('coupon.endTime')" min-width="160">
              <template slot-scope="scope">
                <span class="table-cell-text">{{ scope.row.endTime || '-' }}</span>
              </template>
            </el-table-column>

            <el-table-column prop="number" :label="$t('coupon.stock')" sortable min-width="100">
              <template slot-scope="scope">
                <span class="table-cell-text">{{ scope.row.number || '0' }}</span>
              </template>
            </el-table-column>
            <el-table-column prop="collar" label="限领" sortable min-width="100">
              <template slot-scope="scope">
                <span class="table-cell-text">{{ scope.row.collar || '1' }}</span>
              </template>
            </el-table-column>

            <el-table-column fixed="right" align="center" :label="$i18n.t('crud.menu')" min-width="230">
              <template slot-scope="scope">
                <div class="text-btn-con">
                  <div class="default-btn text-btn" v-if="isAuth('platform:voucher:update')"
                    @click="addOrUpdateHandle(scope.row.id)">修改</div>
                  <!-- <div class="default-btn text-btn" @click="addOrUpdateHandle(scope.row.id)">{{ $t("seckill.view")
                  }}
                  </div> -->
                  <!-- <div class="default-btn text-btn"
                    v-if="isAuth('platform:voucher:update') && (scope.row.putonStatus < 2 || scope.row.putonStatus === 4) && scope.row.overdueStatus === 1"
                    @click="offlineCouponHandle(scope.row)">{{ $t("product.violation") }}</div>
                  <div class="default-btn text-btn"
                    v-if="isAuth('platform:voucher:update') && scope.row.putonStatus > 1 && scope.row.putonStatus < 4"
                    @click="couponAuditHandle(scope.row.couponId)">{{
                      scope.row.putonStatus === 2
                      ? $t("prodList.offShelfManage")
                      : $t("coupon.checkPending")
                    }}</div> -->
                  <div class="default-btn text-btn" @click="deleteHandle(scope.row.id)">{{ $t("coupon.delete") }}
                  </div>
                </div>
              </template>
            </el-table-column>

            <!-- <template slot-scope="scope" slot="menu">
        <el-button
          icon="el-icon-edit"
          type="text"
          size="small"
          @click="addOrUpdateHandle(scope.row.couponId)"
        >{{$t("coupon.viewDetails")}}</el-button>
        <el-button
          icon="el-icon-bottom"
          size="small"
          type="text"
          v-if="isAuth('platform:coupon:update') && scope.row.putonStatus<2"
          @click="offlineCouponHandle(scope.row)"
        >{{$t("coupon.offlineActivities")}}</el-button>
        <el-button
          icon="el-icon-document-checked"
          size="small"
          type="text"
          v-if="isAuth('platform:coupon:update') && scope.row.putonStatus > 1"
          @click="couponAuditHandle(scope.row.couponId)"
        >{{scope.row.putonStatus === 2?$t("coupon.violationOffShelf"):$t("coupon.checkPending")}}</el-button>
        <el-button
          icon="el-icon-delete"
          size="small"
          type="text"
          style="color:red"
          @click="deleteHandle(scope.row.couponId)"
        >{{$t("coupon.delete")}}</el-button>
      </template> -->

          </el-table>
        </div>
        <!-- 分页 -->
        <el-pagination v-if="dataList.length" @size-change="handleSizeChange" @current-change="handleCurrentChange"
          :current-page="page.currentPage" :page-sizes="[10, 20, 50, 100]" :page-size="page.pageSize"
          layout="total, sizes, prev, pager, next, jumper" :total="page.total">
        </el-pagination>
      </div>
      <!-- 表格主体end -->
    </div>
    <!-- 商品审核弹窗 -->
    <CouponAudit v-if="couponAuditVisible" ref="couponAudit" selectUrl="/platform/coupon/getOfflineHandleEventByCouponId"
      updateUrl="/platform/voucher/auditCoupon" @refreshDataList="refreshChange"></CouponAudit>
    <!-- 下线弹窗 -->
    <el-dialog :title="this.$i18n.t('text.tips')" :visible.sync="offlineDialogVisible" append-to-body
      :close-on-click-modal="false" destroy-on-close class="remarks">
      <el-input v-model="offlineReason" type="textarea" :rows="2" maxlength="200"
        :placeholder="this.$i18n.t('coupon.offlineReasonTips')" />
      <span slot="footer" class="dialog-footer">
        <div class="default-btn" @click="offlineDialogVisible = false">{{ $t('remindPop.cancel') }}</div>
        <div class="default-btn primary-btn" @click="offlineSubmit">{{ $t('shopProcess.submit') }}</div>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import CouponAudit from '@/components/offline-audit'
export default {
  data() {
    return {
      theData: null, // 保存上次点击查询的请求条件
      theParams: null, // 保存上次点击查询的请求条件
      search: {
        slot: ''
      },
      dataForm: {
        orderNumber: '',
        status: null
      },
      dateRange: [],
      dataList: [],
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      couponAuditVisible: false,
      isSubmit: false,
      offlineDialogVisible: false, // 下线提示弹窗
      offlineReason: '', // 下线理由
      couponId: '', // 当前操作的优惠券id
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      // 头部搜索表单
      searchForm: {
        name: null
      }
    }
  },
  components: {
    CouponAudit
  },
  mounted() {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList(page, newData = false) {
      this.dataListLoading = true
      if (newData || !this.theData) {
        this.theParams = JSON.parse(JSON.stringify(this.searchForm))
        this.theData = {
          current: page == null ? this.page.currentPage : page.currentPage,
          size: page == null ? this.page.pageSize : page.pageSize,
          'name': this.dataForm.name,
          'startTime': this.dateRange === null ? null : this.dateRange[0], // 开始时间
          'endTime': this.dateRange === null ? null : this.dateRange[1] // 结束时间
        }
      } else {
        this.theData.current = page == null ? this.page.currentPage : page.currentPage
        this.theData.size = page == null ? this.page.pageSize : page.pageSize
      }
      this.$http({
        url: this.$http.adornUrl('/platform/search/prod/get/list'),
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
    orderStatus() {
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.$router.push({
        path: '/marketing-new-voucher',
        query: {
          voucherId: id
        }
      })
    },
    // 删除
    deleteHandle(id) {
      this.$confirm(this.$t('coupon.Tips2') + `${id ? this.$t('coupon.delete') : this.$t('coupon.batchDelete')}` + this.$t('coupon.Tips3'), this.$t('coupon.TipsTitle'), {
        confirmButtonText: this.$t('coupon.confirm'),
        cancelButtonText: this.$t('coupon.cancel'),
        type: 'warning'
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl('/platform/search/prod/update'),
            method: 'post',
            data: this.$http.adornData({
              'id': id,
              'isDeleted': 1,
            })
          }).then(({ data }) => {
            this.$message({
              message: this.$t('publics.operation'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.refreshChange()
              }
            })
          })
        })
        .catch(() => { })
    },
    // 条件查询
    searchChange(newData = false) {
      this.page.currentPage = 1
      this.getDataList(this.page, newData)
    },
    // 刷新回调用
    refreshChange() {
      this.getDataList(this.page)
    },
    // 多选变化
    selectionChange(val) {
      this.dataListSelections = val
    },
    // 清空自定义数据
    searchReset() {
      this.dateRange = []
    },
    // 下线活动
    offlineCouponHandle(row) {
      this.$prompt(this.$t('coupon.Tips0') + row.couponName + this.$t('coupon.Tips1'), this.$t('coupon.TipsTitle'), {
        confirmButtonText: this.$i18n.t('coupon.confirm'),
        cancelButtonText: this.$i18n.t('coupon.cancel'),
        inputPattern: /\s\S+|S+\s|\S/, // 不能全为空格 空字符串
        inputErrorMessage: this.$i18n.t('coupon.offlineReasonNotEmpty'),
        inputValidator: (value) => {
          if (value.length > 100) {
            return this.$i18n.t('seckill.offlineReasonTip1')
          }
        }
      }).then((obj) => {
        this.$http({
          url: this.$http.adornUrl(`/platform/coupon/offline`),
          method: 'post',
          data: this.$http.adornData({
            handleId: row.couponId,
            offlineReason: obj.value || null
          })
        }).then(({ data }) => {
          this.$message({
            message: this.$t('publics.operation'),
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.getDataList(this.page)
            }
          })
        })
      })

      // this.offlineReason = ''
      // this.couponId = row.couponId
      // this.offlineDialogVisible = true
    },
    // 提交下线操作
    offlineSubmit() {
      if (this.isSubmit) {
        return
      }
      this.isSubmit = true
      if (!this.offlineReason.trim() || this.offlineReason.trim().length > 200) {
        this.$message({
          message: this.$i18n.t('coupon.offlineReasonNotEmpty'),
          type: 'error',
          duration: 1000
        })
        this.isSubmit = false
        return
      }
      this.$http({
        url: this.$http.adornUrl(`/platform/coupon/offline`),
        method: 'post',
        data: this.$http.adornData({
          handleId: this.couponId,
          offlineReason: this.offlineReason
        })
      }).then(({ data }) => {
        this.$message({
          message: this.$i18n.t('publics.operation'),
          type: 'success',
          duration: 1500,
          onClose: () => {
            this.offlineDialogVisible = false
            this.isSubmit = false
            this.getDataList(this.page)
          }
        })
      }).catch((e) => {
        this.isSubmit = false
      })
    },
    // 弹窗处理
    couponAuditHandle(id) {
      this.couponAuditVisible = true
      this.$nextTick(() => {
        this.$refs.couponAudit.init(id)
      })
    },
    clearSearch() {
      this.dateRange = []
      this.searchForm.shopName = null
      this.searchForm.couponName = null
      this.searchForm.overdueStatus = null
      this.searchForm.putonStatus = null
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
