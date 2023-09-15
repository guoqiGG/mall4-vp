<template>
  <div class="mod-marketing-coupon">
    <!-- 新版规范 -->
    <div class="coupon-mod">
      <!-- 表格主体 -->
      <div class="main-container">
        <div class="operation-bar">
          <div class="default-btn primary-btn" v-if="isAuth('platform:cardsandcoupons:save')"
            @click="addOrUpdateHandle()">{{
              $t("crud.addTitle") }}
          </div>
        </div>
        <!-- 表格 -->
        <div class="table-con seckill-table">
          <el-table :data="dataList" header-cell-class-name="table-header" row-class-name="table-row-low"
            style="width: 100%">
            <el-table-column prop="stockId" label="批次号"></el-table-column>
            <el-table-column prop="stockName" label="名称"></el-table-column>
            <el-table-column prop="status" label="是否授权">
              <template slot-scope="scope">
               <div style="color:#155BD7;" v-if="scope.row.status==1">已授权</div>
               <div v-else>未授权</div>
              </template>
            </el-table-column>

            <el-table-column prop="createTime" label="创建时间"></el-table-column>
            <el-table-column fixed="right" align="center" :label="$i18n.t('crud.menu')" min-width="100">
              <template slot-scope="scope">
                <div class="text-btn-con">
                  <div class="default-btn text-btn" @click="viewHandle(scope.row.stockId)">查看</div>
                  <div class="default-btn text-btn" v-if="scope.row.status==0" @click="authorizeHandle(scope.row.stockId)">授权</div>
                </div>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </div>
      <!-- 分页 -->
      <el-pagination v-if="dataList.length" @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="page.currentPage" :page-sizes="[10, 20, 50, 100]" :page-size="page.pageSize"
        layout="total, sizes, prev, pager, next, jumper" :total="page.total">
      </el-pagination>
      <!-- 表格主体end -->
    </div>

    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="refreshChange"></add-or-update>
    <!-- 查看 -->
    <view-coupons v-if="viewCouponsVisible" ref="viewCoupons"></view-coupons>

  </div>
</template>

<script>
import AddOrUpdate from './cardsandcoupons-add-or-update'
import ViewCoupons from './cardsandcoupons-view'
export default {
  data() {
    return {
      theData: null, // 保存上次点击查询的请求条件
      dataList: [],
      dataListLoading: false,
      addOrUpdateVisible: false,
      viewCouponsVisible: false,
      isSubmit: false,
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
    }
  },
  components: {
    AddOrUpdate,
    ViewCoupons
  },
  mounted() {
    this.getDataList()
  },
  methods: {
    // 卡券授权
    authorizeHandle(stock_id) {
      this.$http({
        url: this.$http.adornUrl('/platform/shopCompany/update/wx/coupon/status'),
        method: 'get',
        params: this.$http.adornParams({
          stockId: stock_id
        })
      }).then(({ data }) => {
        console.log(data)
        if (data.build_time) {
          this.$message({
            message: '授权成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.getDataList()
            }
          })
        }
      })
    },
    // 获取数据列表
    getDataList(page) {
      this.dataListLoading = true
      this.theData = {
        current: page == null ? this.page.currentPage : page.currentPage,
        size: page == null ? this.page.pageSize : page.pageSize
      }
      this.$http({
        url: this.$http.adornUrl('/platform/shopCompany/wx/coupon/list'),
        method: 'get',
        params: this.$http.adornParams(
          Object.assign(this.theData), false
        )
      }).then(({ data }) => {
        console.log(data)
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
    // 新增 / 修改
    addOrUpdateHandle(val) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(val)
      })
    },
    // 查看
    viewHandle(val) {
      this.viewCouponsVisible = true
      this.$nextTick(() => {
        this.$refs.viewCoupons.init(val)
      })
    },

    // 刷新回调用
    refreshChange() {
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
