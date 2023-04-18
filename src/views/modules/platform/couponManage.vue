<template>
  <div class="mod-marketing-coupon">
    <div class="search-bar">
      <el-form @submit.native.prevent :inline="true" class="search-form" ref="searchForm" :model="searchForm" size="small" >
        <div class="input-row">
          <el-form-item prop="couponName" :label="$t('coupon.couponName') + ':'" label-width="80px" class="search-form-item">
            <el-input type="text" v-model="searchForm.couponName" :placeholder="$t('coupon.couponName')"></el-input>
          </el-form-item>
          <el-form-item prop="overdueStatus" :label="$t('coupon.expiredStatus') + ':'" label-width="60px" class="search-form-item">
            <el-select v-model="searchForm.overdueStatus" :placeholder="$t('coupon.expiredStatus')">
              <el-option :label="$t('coupon.notExpired')" :value="1"></el-option>
              <el-option :label="$t('coupon.expired')" :value="0"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item prop="putonStatus" :label="$t('coupon.deliveryStatus') + ':'" label-width="80px"  class="search-form-item">
            <el-select v-model="searchForm.putonStatus" :placeholder="$t('coupon.deliveryStatus')">
              <el-option :label="$t('coupon.cancelLaunch')" :value="-1"></el-option>
              <el-option :label="$t('coupon.launched')" :value="1"></el-option>
              <el-option :label="$t('coupon.waitAutoLaunch')" :value="0"></el-option>
              <el-option :label="$t('coupon.waitLaunch')" :value="4"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item
            :label="$t('coupon.expire')+':'"
            label-width="70px"
          >
            <el-date-picker
              v-model="dateRange"
              type="datetimerange"
              :range-separator="$t('coupon.to')"
              value-format="yyyy-MM-dd HH:mm:ss"
              :start-placeholder="$t('coupon.startTime')"
              :end-placeholder="$t('coupon.endTime')"
            ></el-date-picker>
          </el-form-item>
          <div class="default-btn primary-btn marginBottom" @click="searchChange(true)">{{$t('product.search')}}</div>
          <div class="default-btn" @click="resetSearchForm('searchForm')">{{$t('product.reset')}}</div>
        </div>
      </el-form>
    </div>
    <div class="main-container">
      <div class="operation-bar">
        <div
          class="default-btn primary-btn"
          @click.stop="addOrUpdateHandle()">
          {{ $t('sysManagement.add') }}
        </div>
      </div>
      <div class="table-con coupon-table">
        <el-table
          ref="couponListTable"
          :data="dataList"
          header-cell-class-name="table-header"
          row-class-name="table-row-low"
          style="width: 100%">

          <el-table-column
            fixed
            align="left"
            prop="couponName"
            :label="$t('coupon.couponName')"
            width="200">
            <template slot-scope="scope">
              <span class="table-cell-text">{{ scope.row.couponName || '-' }}</span>
            </template>
          </el-table-column>

          <el-table-column
            align="left"
            prop="couponType"
            :label="$t('coupon.couponType')"
            width="100">
            <template slot-scope="scope">
              <span v-if="scope.row.couponType === 1" class="tag-text">{{$t("coupon.cashCoupon")}}</span>
              <span v-else-if="scope.row.couponType === 2" class="tag-text">{{$t("coupon.discountVoucher")}}</span>
              <span v-else-if="scope.row.couponType === 3" class="tag-text">{{$t("coupon.coinCertificate")}}</span>
            </template>
          </el-table-column>

          <el-table-column
            prop="startTime"
            :label="$t('coupon.startTime')"
            width="160">
            <template slot-scope="scope">
              <span class="table-cell-text">{{ scope.row.startTime || '-' }}</span>
            </template>
          </el-table-column>
          <el-table-column
            prop="endTime"
            :label="$t('coupon.endTime')"
            width="160">
            <template slot-scope="scope">
              <span class="table-cell-text">{{ scope.row.endTime || '-' }}</span>
            </template>
          </el-table-column>

          <el-table-column
            prop="overdueStatus"
            :label="$t('coupon.expiredStatus')"
            width="auto">
            <template slot-scope="scope">
              <span v-if="scope.row.overdueStatus === 1" class="tag-text">{{$t("coupon.notExpired")}}</span>
              <span v-else class="tag-text">{{$t("coupon.expired")}}</span>
            </template>
          </el-table-column>

          <el-table-column
            prop="putonStatus"
            :label="$t('coupon.deliveryStatus')"
            width="auto">
            <template slot-scope="scope">
              <span v-if="scope.row.putonStatus === 1" class="tag-text">{{$t("coupon.launched")}}</span>
              <span
                v-else-if="scope.row.putonStatus === 0"
                class="tag-text"
              >{{$t("coupon.waitAutoLaunch")}}</span>
              <span
                v-else-if="scope.row.putonStatus === -1"
                class="tag-text"
              >{{$t("coupon.cancelLaunch")}}</span>
              <span
                v-else-if="scope.row.putonStatus ===2"
                class="tag-text"
              >{{$t("coupon.violationOffShelf")}}</span>
              <span
                v-else-if="scope.row.putonStatus ===3"
                class="tag-text"
              >{{$t("coupon.waitingForReview")}}</span>
              <span
                v-else-if="scope.row.putonStatus ===4"
                class="tag-text"
              >{{$t("coupon.waitLaunch")}}</span>
            </template>
          </el-table-column>

          <el-table-column
            prop="launchTime"
            :label="$t('coupon.timeToMarket')"
            width="160"
          >
            <template slot-scope="scope">
              <span class="table-cell-text">{{ scope.row.launchTime || '-' }}</span>
            </template>
          </el-table-column>

          <el-table-column
            prop="stocks"
            :label="$t('coupon.stock')"
            width="auto">
            <template slot-scope="scope">
              <span class="table-cell-text">{{ scope.row.stocks || '-' }}</span>
            </template>
          </el-table-column>

          <el-table-column
            prop="takeNum"
            :label="$t('dataAnalysis.receiveTimes')"
            width="auto">
            <template slot-scope="scope">
              <span class="table-cell-text">{{ scope.row.takeNum || '-' }}</span>
            </template>
          </el-table-column>

          <el-table-column
            prop="useNum"
            :label="$t('dataAnalysis.miniMallUsedTimes')"
            width="120">
            <template slot-scope="scope">
              <span class="table-cell-text">{{ scope.row.useNum || '-' }}</span>
            </template>
          </el-table-column>

          <el-table-column
            prop="getWay"
            :label="$t('coupon.getWay')"
            width="auto">
            <template slot-scope="scope">
              <span class="tag-text">{{[$t("coupon.receiveDirectly"),$t("coupon.exchangeOrSystemIssue")][scope.row.getWay]}}</span>
            </template>
          </el-table-column>

          <el-table-column
            fixed="right"
            align="center"
            :label="$t('publics.operating')"
            width="200"
            >
            <template slot-scope="scope">
              <div class="text-btn-con">
                <div
                  class="default-btn text-btn"
                  @click="addOrUpdateHandle(scope.row.couponId)"
                >{{$t("coupon.edit")}}</div>
                <div
                  class="default-btn text-btn"
                  v-if="scope.row.putonStatus > 1 && scope.row.putonStatus !== 4"
                  @click="auditEventHandle(scope.row.couponId)"
                >{{$t("coupon.offlineActivities")}}</div>
                <div
                  class="default-btn text-btn"
                  @click="deleteHandle(scope.row.couponId)"
                >{{$t("coupon.delete")}}</div>
              </div>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <el-pagination
        v-if="dataList.length"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="page.currentPage"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="page.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="page.total">
      </el-pagination>
    </div>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="refreshChange"></add-or-update>
    <!-- 下线管理弹窗-->
    <offlineEventHandle
      v-if="offlineEventHandleVisible"
      selectUrl="/admin/coupon/getOfflineHandleEventByCouponId"
      applyUrl="/admin/coupon/auditApply"
      ref="offlineEvent"
      @refreshDataList="refreshChange"
    ></offlineEventHandle>
  </div>
</template>

<script>
import AddOrUpdate from './couponManage-add-or-update'
import OfflineEventHandle from '@/components/offline-event-handle'
export default {
  data () {
    return {
      theData: null, // 保存上次点击查询的请求条件
      theParams: null, // 保存上次点击查询的请求条件

      search: {
        slot: ''
      },
      dataForm: {
        shopId: 0,
        orderNumber: '',
        status: null
      },
      dateRange: [],
      dataList: [],
      dataListLoading: false,
      offlineEventHandleVisible: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      searchForm: {
        couponName: '',
        overdueStatus: '',
        putonStatus: ''
      }
    }
  },
  components: {
    AddOrUpdate,
    OfflineEventHandle
  },
  created () {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList (page, newData = false) {
      this.dataListLoading = true
      if (this.page) {
        let size = Math.ceil(this.page.total / this.page.pageSize)
        this.page.currentPage = (this.page.currentPage > size ? size : this.page.currentPage) || 1
      }
      if (newData || !this.theData) {
        this.theParams = JSON.parse(JSON.stringify(this.searchForm))
        this.theData = {
          current: page == null ? this.page.currentPage : page.currentPage,
          size: page == null ? this.page.pageSize : page.pageSize,
          'couponName': this.dataForm.couponName,
          'shopId': this.dataForm.shopId,
          'overdueStatus': this.dataForm.overdueStatus,
          'putonStatus': this.dataForm.putonStatus,
          'startTime': this.dateRange === null ? null : this.dateRange[0], // 开始时间
          'endTime': this.dateRange === null ? null : this.dateRange[1] // 结束时间
        }
      } else {
        this.theData.current = page == null ? this.page.currentPage : page.currentPage
        this.theData.size = page == null ? this.page.pageSize : page.pageSize
      }

      this.$http({
        url: this.$http.adornUrl('/platform/coupon/page'),
        method: 'get',
        params: this.$http.adornParams(
          Object.assign(
            this.theData,
            this.theParams
          ), false
        )
      }).then(({ data }) => {
        this.dataList = data.records
        this.page.total = data.total
        this.dataListLoading = false
      })
    },
    orderStatus () {

    },
    // 新增 / 修改
    addOrUpdateHandle (val) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(val)
      })
    },
    // 删除
    deleteHandle (id) {
      this.$confirm(this.$i18n.t('remindPop.makeSure') + ' ' + `[${id ? this.$i18n.t('remindPop.delete') : this.$i18n.t('remindPop.batchDeletion')}]` + ' ' + this.$i18n.t('remindPop.operation') + '?', this.$i18n.t('remindPop.remind'), {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl('/platform/coupon'),
            method: 'delete',
            data: this.$http.adornData(id, false)
          }).then(({ data }) => {
            this.page.total = this.page.total - 1
            this.$message({
              message: this.$i18n.t('remindPop.success'),
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
    searchChange (newData = false) {
      this.getDataList(this.page, newData)
    },
    // 刷新回调用
    refreshChange () {
      this.getDataList(this.page)
    },
    // 多选变化
    selectionChange (val) {
      this.dataListSelections = val
    },
    // 清空自定义数据
    searchReset () {
      this.dateRange = []
    },
    // 下线管理
    auditEventHandle (id) {
      this.offlineEventHandleVisible = true
      this.$nextTick(() => {
        this.$refs.offlineEvent.init(id)
      })
    },
    /**
     * 重置表单
     * @param {String} formName 表单名称
     */
    resetSearchForm (formName) {
      this.$refs[formName].resetFields()
      this.searchReset()
    },

    handleSizeChange (val) {
      this.page.pageSize = val
      this.getDataList()
    },
    handleCurrentChange (val) {
      this.page.currentPage = val
      this.getDataList()
    }
  }
}
</script>
<style lang="scss" scoped>
.marginBottom{margin-bottom:20px}
</style>
