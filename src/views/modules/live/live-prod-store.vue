<template>
  <div class="mod-live-liveProdStore">
    <!-- 搜索栏 -->
    <div class="search-bar">
      <el-form @submit.native.prevent :inline="true" class="search-form" ref="searchForm" :model="searchForm"
        size="small">
        <div class="input-row">
          <el-form-item prop="name" :label="$t('product.prodName') + ':'">
            <el-input v-model="searchForm.name" type="text" clearable :placeholder="$t('product.prodName')"></el-input>
          </el-form-item>
          <el-form-item>
            <div class="default-btn primary-btn" @click="searchChange(true)">{{ $t('crud.searchBtn') }}</div>
            <div class="default-btn" @click="resetForm('searchForm')">{{ $t('product.reset') }}</div>
          </el-form-item>
        </div>
      </el-form>
    </div>
    <div class="main-container">
        <!-- 搜索栏end -->
        <div class="operation-bar">
        <div class="default-btn primary-btn" @click="addOrUpdateHandle(0, 1)">
          新增
        </div>
        <span class="live-tips">
          （今日剩余提交审核直播商品次数：<span class="stress">{{shopNum}}</span>次，平台今日剩余限额<span class="stress">{{ totalNum }}</span>次）</span>
      </div>
      <div class="table-con">
        <div class="tips">
          <span
            class="text">{{ $t('liveRoom.newLiveProdNum') }}{{ $t('liveRoom.platformRemainingQuota') }}{{ totalNum }}{{ $t('user.bout') }}</span>
        </div>
        <el-table :data="dataList" header-cell-class-name="table-header" row-class-name="table-row" style="width: 100%">
          <el-table-column :label="$t('brand.serialNumber')" type="index" width="80" />
          <el-table-column align="left" prop="coverPic" :label="$t('group.prodInfo')" width="400">
            <template slot-scope="scope">
              <div class="table-cell-con">
                <div class="table-cell-image">
                  <ImgShow :src="scope.row.coverPic" />
                </div>
                <span class="table-cell-text">{{ scope.row.name }}</span>
              </div>
            </template>
          </el-table-column>
          <el-table-column prop="price" :label="$t('product.price')" />
          <el-table-column prop="status" :label="$t('product.status')">
            <template slot-scope="scope">
              <span v-if="scope.row.status === -1">{{ $t('liveRoom.goodsDeleted') }}</span>
              <span v-if="scope.row.status === 0">{{ $t('liveRoom.notReviewed') }}</span>
              <span v-if="scope.row.status === 1">{{ $t('liveRoom.underReview') }}</span>
              <span v-if="scope.row.status === 2">{{ $t('liveRoom.approved') }}</span>
              <span v-if="scope.row.status === 3">{{ $t('liveRoom.auditFailed') }}</span>
              <span v-if="scope.row.status === 4">{{ $t('product.violation') }}</span>
              <span v-if="scope.row.status === 5">{{ $t('scoreProdList.platformOffShelf') }}</span>
            </template>
          </el-table-column>
          <el-table-column width="150" prop="createTime" :label="$t('liveRoom.addGoodsTime')">
            <template slot-scope="scope">
              <span class="table-cell-text">{{ scope.row.createTime || '-' }}</span>
            </template>
          </el-table-column>
          <el-table-column width="150" prop="verifyTime" :label="$t('liveRoom.submissionTime')">
            <template slot-scope="scope">
              <span class="table-cell-text">{{ scope.row.verifyTime || '-' }}</span>
            </template>
          </el-table-column>
          <el-table-column width="150" prop="successTime" :label="$t('liveRoom.passTime')">
            <template slot-scope="scope">
              <span class="table-cell-text">{{ scope.row.successTime || '-' }}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" :label="$t('crud.menu')" width="100" fixed="right">
            <template slot-scope="scope">
              <div class="text-btn-con">
                <div :class="[scope.row.status === 1 ? 'disabled-btn' : '', 'default-btn text-btn']"
                  @click="deleteHandle(scope.row.liveProdStoreId, scope.row.status)">{{ $t('remindPop.delete') }}</div>
              </div>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="page.currentPage" :page-sizes="[10, 20, 50, 100]" :page-size="page.pageSize" :total="page.total"
        layout="total, sizes, prev, pager, next, jumper"></el-pagination>
    </div>
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="refreshChange"
    />
  </div>
</template>

<script>
import AddOrUpdate from './liveProdStore-add-or-update.vue'

export default {
  data () {
    return {
      theData: null, // 保存上次点击查询的请求条件

      dataList: [],
      shopNum: 0,
      totalNum: 0,
      deleteShopNum: 0,
      deleteTotalNum: 0,
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      searchForm: {
        name: ''
      }, // 搜索
      dataListLoading: false,
      resourcesUrl: process.env.VUE_APP_RESOURCES_URL,
      addOrUpdateVisible: false
    }
  },
  components: {
    AddOrUpdate
  },
  created () {
  },
  mounted () {
    this.getDataList(this.page)
  },
  methods: {
    getDataList (page, newData = false) {
      this.dataListLoading = true
      if (this.page) {
        let size = Math.ceil(this.page.total / this.page.pageSize)
        this.page.currentPage = (this.page.currentPage > size ? size : this.page.currentPage) || 1
      }
      this.getAddProdNum()
      if (newData || !this.theData) {
        this.theData = JSON.parse(JSON.stringify(this.searchForm))
      }
      this.$http({
        url: this.$http.adornUrl('/live/liveProdStore/page'),
        method: 'get',
        params: this.$http.adornParams(
          Object.assign(
            {
              current: page == null ? this.page.currentPage : page.currentPage,
              size: page == null ? this.page.pageSize : page.pageSize
            },
            this.theData
          )
        )
      }).then(({ data }) => {
        this.dataList = data.records
        this.page.total = data.total
        this.dataListLoading = false
      })
    },
    // 获取剩余可新增得数量
    getAddProdNum () {
      this.$http({
        url: this.$http.adornUrl('/live/liveProdStore/getAddProdNum'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({ data }) => {
        this.shopNum = data.shopNum
        this.totalNum = data.totalNum
        this.deleteShopNum = data.deleteShopNum
        this.deleteTotalNum = data.deleteTotalNum
      })
    },
    // 新增 / 修改
    addOrUpdateHandle (id, type) {
      // type = 1 新增 type = 2 修改  type = 3 查看
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id, type)
      })
    },
    /**
     * 提交审核
     */
    submitVerify (id) {
      this.$confirm(this.$i18n.t('liveRoom.auditNeeds') + ' 1 - 7 ' + this.$i18n.t('liveRoom.day') + '，' + this.$i18n.t('liveRoom.confirmSubmit') + '？', this.$i18n.t('liveRoom.submitReview'), {
        confirmButtonText: this.$i18n.t('remindPop.confirm'),
        cancelButtonText: this.$i18n.t('remindPop.cancel'),
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/live/liveProdStore/submitVerify'),
          method: 'put',
          data: this.$http.adornData({
            liveProdStoreId: id
          })
        }).then(({ data }) => {
          if (data) {
            this.$message({
              message: this.$i18n.t('liveRoom.submitReview'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.refreshChange()
              }
            })
          }
        })
      }).catch(() => { })
    },
    deleteHandle (id, status) {
      if (status === 1) {
        return
      }
      this.$confirm(this.$i18n.t('liveRoom.delTips') + ' ' + this.deleteTotalNum + this.$i18n.t('distribution.bcTip4') + ',' + this.$i18n.t('seckill.isDeleOper'), this.$i18n.t('remindPop.remind'), {
        confirmButtonText: this.$i18n.t('remindPop.confirm'),
        cancelButtonText: this.$i18n.t('remindPop.cancel'),
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/live/liveProdStore/' + id),
          method: 'delete',
          data: this.$http.adornData({})
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
      }).catch(() => { })
    },
    /**
     * 刷新回调
     */
    refreshChange () {
      this.getDataList(this.page)
    },
    searchChange (newData = false) {
      this.page.currentPage = 1
      this.getDataList(this.page, newData)
    },
    handleSizeChange (val) {
      this.page.pageSize = val
      this.getDataList(this.page)
    },
    handleCurrentChange (val) {
      this.page.currentPage = val
      this.getDataList(this.page)
    },
    resetForm (formName) {
      this.$refs[formName].resetFields()
      this.dateRange = []
    }
  }
}
</script>
<style lang="scss" scoped>
.mod-live-liveProdStore {
  .tips {
    margin-bottom: 20px;

    .text {
      color: #FF0000;
    }
  }
}
</style>
