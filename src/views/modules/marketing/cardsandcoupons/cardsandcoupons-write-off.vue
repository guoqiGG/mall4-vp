<template>
  <div class="mod-marketing-coupon">
    <!-- 新版规范 -->
    <div class="coupon-mod">
      <!-- 搜索栏 -->
      <div class="search-bar">
        <el-form @submit.native.prevent :inline="true" class="search-form" ref="test-form" :model="searchForm"
          size="small">
          <div class="input-row">
            <el-form-item label="团长手机号:" class="search-form-item">
              <el-input v-model="searchForm.distributionUserMobile" placeholder="团长手机号"></el-input>
            </el-form-item>
            <el-form-item>
              <div class="default-btn primary-btn" @click="searchChange(true)">{{ $t('shopFeature.searchBar.search') }}
              </div>
              <div class="default-btn" @click="clearSearch">{{ $t('product.reset') }}</div>
              <div class="default-btn" @click="getSoldExcel()">{{ $t("order.export") }}</div>
            </el-form-item>
          </div>
        </el-form>
      </div>
      <!-- 搜索栏end -->
      <!-- 表格主体 -->
      <div class="main-container">
        <!-- 表格 -->
        <div class="table-con seckill-table">
          <el-table :data="dataList" header-cell-class-name="table-header" row-class-name="table-row-low"
            style="width: 100%">
            <el-table-column prop="name" label="名称"></el-table-column>
            <el-table-column prop="userName" label="用户昵称"></el-table-column>
            <el-table-column prop="realName" label="团长"></el-table-column>
            <el-table-column prop="distributionUserMobile" label="团长手机号"></el-table-column>
            <el-table-column prop="remark" label="备注"></el-table-column>
            <el-table-column prop="createTime" label="核销时间"></el-table-column>
          </el-table>
        </div>
      </div>
      <!-- 分页 -->
      <el-pagination v-if="dataList.length" @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="page.currentPage" :page-sizes="[2, 10, 20, 50, 100]" :page-size="page.pageSize"
        layout="total, sizes, prev, pager, next, jumper" :total="page.total">
      </el-pagination>
      <!-- 表格主体end -->
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
      dataListLoading: false,
      isSubmit: false,
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      // 头部搜索表单
      searchForm: {
        distributionUserMobile: ''
      }
    }
  },
  mounted() {
    this.getDataList()
  },
  methods: {
    // 订单导出
    getSoldExcel() {
      // if (!this.searchForm.distributionUserMobile) {
      //   this.$message.error(this.$i18n.t('请输入需要导出的团长手机号'))
      //   return
      // }
      this.$confirm(this.$i18n.t('order.exportReport'), this.$i18n.t('remindPop.remind'), {
        confirmButtonText: this.$i18n.t('remindPop.confirm'),
        cancelButtonText: this.$i18n.t('remindPop.cancel'),
        type: 'warning'
      }).then(() => {
        const loading = this.$loading({
          lock: true,
          target: '.mod-order-order',
          customClass: 'export-load',
          background: 'transparent',
          text: '核销记录导出'
        })
        this.$http({
          url: this.$http.adornUrl('/platform/order/distribution/card/write/soldExcel'),
          method: 'get',
          params: this.$http.adornParams({
            'distributionUserMobile': this.searchForm.distributionUserMobile,

          }),
          responseType: 'blob' // 解决文件下载乱码问题
        }).then(({ data }) => {
          loading.close()
          var blob = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8' })
          const fileName = '核销记录导出'
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
      })
    },
    // 获取数据列表
    getDataList(page, newData = false) {
      this.dataListLoading = true
      if (newData || !this.theData) {
        this.theParams = JSON.parse(JSON.stringify(this.searchForm))
        this.theData = {
          current: page == null ? this.page.currentPage : page.currentPage,
          size: page == null ? this.page.pageSize : page.pageSize,
          distributionUserMobile: this.searchForm.distributionUserMobile
        }
      } else {
        this.theData.current = page == null ? this.page.currentPage : page.currentPage
        this.theData.size = page == null ? this.page.pageSize : page.pageSize
      }
      this.$http({
        // url: this.$http.adornUrl('/platform/shopCompany/get/wx/marketing/coupon/detail'),
        url: this.$http.adornUrl('/platform/shopCompany/get/user/coupon/list'),
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
    // 条件查询
    searchChange(newData = false) {
      this.page.currentPage = 1
      this.getDataList(this.page, newData)
    },
    clearSearch() {
      this.searchForm.distributionUserMobile = ''
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
