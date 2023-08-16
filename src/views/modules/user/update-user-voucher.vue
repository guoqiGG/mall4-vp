<template>
  <div>
    <el-dialog title="选择礼品券" :visible.sync="visible" width="50%" class="select-voucher-dialog">
      <el-form @submit.native.prevent :inline="true" :model="searchForm" class="demo-form-inline form">
        <el-form-item>
          <el-input v-model="name" size="small" placeholder="请输入礼品券名称搜索" clearable></el-input>
        </el-form-item>
        <el-form-item>
          <div class="default-btn primary-btn" @click="searchChange">{{ $t("pictureManager.query") }}</div>
        </el-form-item>
      </el-form>
      <div class="main-container">
        <div class="prods-select-body table-con">
          <el-table ref="voucherTable" :data="dataList" header-cell-class-name="table-header"
            row-class-name="table-row-low" v-loading="dataListLoading" @selection-change="selectChangeHandle"
            style="width: 100%;">
            <el-table-column v-if="isSingle" width="50">
              <template slot-scope="scope">
                <div>
                  <el-radio :label="scope.row.id" v-model="singleSelectVoucherId"
                    @change.native="getSelectProdRow(scope.row)">&nbsp;</el-radio>
                </div>
              </template>
            </el-table-column>
            <el-table-column v-if="!isSingle" type="selection" width="50"></el-table-column>
            <el-table-column prop="name" label="礼品券名称" width="200">
              <template slot-scope="scope">
                <span class="table-cell-text"> {{ scope.row.name }} </span>
              </template>
            </el-table-column>
            <el-table-column prop="number" label="库存数量"></el-table-column>
            <el-table-column prop="collar" label="领券上限"></el-table-column>
            <el-table-column align="center" label="每人发放" width="160">
              <template slot="header">
                <span>每人发放</span>
                <el-popover placement="top" width="200" trigger="hover" content="不超过每人限领，不超过剩余库存">
                  <i class="el-icon-question" slot="reference"></i>
                </el-popover>
              </template>
              <template slot-scope="scope">
                <el-input-number size="mini" v-model="scope.row.eachObtain" controls-position="right"
                  @change="handleChange(scope.row, scope.$index)" :min="scope.row.number > 0 ? 1 : 0"
                  :max="scope.row.number > 0 ? (scope.row.number >= scope.row.collar ? scope.row.collar : scope.row.number) : 0"></el-input-number>
              </template>
            </el-table-column>
          </el-table>
        </div>
        <el-pagination v-if="dataList.length" @size-change="sizeChangeHandle" @current-change="currentChangeHandle"
          :current-page="page.pageIndex" :page-sizes="[10, 20, 50, 100]" :page-size="page.pageSize" :total="page.total"
          layout="total, sizes, prev, pager, next, jumper"></el-pagination>
        <!-- <div v-if="tagList.length < 1">暂无数据</div> -->
      </div>
      <el-alert :title="this.$i18n.t('publics.sendVoucherTips')" type="warning" show-icon :closable="false">
      </el-alert>
      <span slot="footer">
        <div @click="visible = false" class="default-btn">{{ $t('remindPop.cancel') }}</div>
        <div class="default-btn primary-btn" @click="submitProds()">{{ $t('remindPop.confirm') }}</div>
      </span>
    </el-dialog>
  </div>
</template>


<script>
import { Debounce } from '@/utils/debounce'
export default {

  data() {
    return {
      visible: false,
      confirmLoad: false,
      dataForm: {
        userIds: [],
        gifts: []
      },
      dataList: [], // 标签
      dataRule: {
      },
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      dataListLoading: false,
      searchForm: {},
      name: null,
      dataListSelections: []
    }
  },
  components: {
  },
  props: {
    value: {
      default: '',
      type: String
    },
     isSingle: {
      default: false,
      type: Boolean
    },
    // 最大上传数量
    collar: {
      default: 9,
      type: Number
    }
  },
  mounted() {
  },
  methods: {
    init(ids) {
      this.visible = true
      this.dataForm.userIds = ids
      this.getDataList()
    },
    // 每页数
    sizeChangeHandle(val) {
      this.page.pageSize = val
      this.page.currentPage = 1
      this.getDataList(this.page)
    },
    // 当前页
    currentChangeHandle(val) {
      this.page.currentPage = val
      this.getDataList(this.page)
    },
    // 分页获取礼品券列表
    getDataList(page) {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/platform/search/prod/get/list'),
        method: 'get',
        params: this.$http.adornParams(
          Object.assign(
            {
              current: page == null ? this.page.currentPage : page.currentPage,
              size: page == null ? this.page.pageSize : page.pageSize,
            },
            this.searchForm
          )
        )
      }).then(({ data }) => {
        this.dataList = data.records
        this.dataList.forEach(item => {
          this.$set(item, 'eachObtain', 1)
        })
        this.page.total = data.total
        this.dataListLoading = false
      })
    },
    // 搜索
    searchChange() {
      this.searchForm.name = this.name
      this.getDataList(this.page)
    },
    // 单选商品事件
    getSelectProdRow(row) {
      console.log('aa',row)
      this.dataListSelections = [row]
    },
    // 多选
    // 多选点击事件
    selectChangeHandle(selection) {
      console.log('bb',selection)
      this.dataList.forEach((tableItem) => {
        let selectedProdIndex = selection.findIndex((selectedVoucher) => {
          if (!selectedVoucher) {
            this.dataListSelections = []
            return false
          }
          return selectedVoucher.id === tableItem.id
        })
        let dataSelectedProdIndex = this.dataListSelections.findIndex((dataselectedVoucher) => dataselectedVoucher.id === tableItem.id)
        if (selectedProdIndex > -1 && dataSelectedProdIndex === -1) {
          this.dataListSelections.push(tableItem)
        } else if (selectedProdIndex === -1 && dataSelectedProdIndex > -1) {
          this.dataListSelections.splice(dataSelectedProdIndex, 1)
        }
      })
    },
    // 确定事件
    submitProds() {
      // 商品单选的情况
      if (this.isSingle) {
        this.dataListSelections.length && this.$emit('refreshSelectVoucherList', this.dataListSelections[0])
        this.dataListSelections = []
        this.visible = false
        return
      }
      // 多选
      let vouchers = [] 
      if (this.dataListSelections.length < 1) {
      } else {
        this.dataListSelections.forEach(item => {
          let voucherIndex = vouchers.findIndex((voucher) => voucher.giftId === item.id)
          if (voucherIndex === -1) {
            console.log('111',item.id)
            vouchers.push({ giftId: item.id,  eachObtain: item.eachObtain })
          }
        })
      }
      
      this.dataForm.gifts = vouchers
      this.dataListSelections = []
      this.confirm()
    },
    confirm: Debounce(function () {
      if (!this.dataForm.userIds) {
        return
      }
      if (!this.dataForm.gifts || !this.dataForm.gifts.length) {
        return this.$message.error('请选择礼品券')
      }
      this.confirmLoad = true
      let vouchers = []
      this.dataForm.gifts.forEach(element => {
        console.log(222,element.id)
        let obj = {}
        obj.giftId = element.giftId
        obj.eachObtain = element.eachObtain
        vouchers.push(obj)
      })
      this.dataForm.gifts = vouchers
      let param = this.dataForm
      console.log(param)
      this.$http({
        url: this.$http.adornUrl(`/platform/search/prod/user/gift/add`),
        method: 'post',
        data: this.$http.adornData(param)
      }).then(({ data }) => {
        this.active = []
        this.$message({
          message: this.$i18n.t('remindPop.success'),
          type: 'success',
          duration: 1500,
          onClose: () => {
            this.visible = false
            this.confirmLoad = false
            this.$emit('refreshDataList', this.page)
          }
        })
      }).catch((e) => {
      })
    }, 1000),
    handleChange(row, index) {
      this.$set(this.dataList, index, this.dataList[index])
    }
  }
}
</script>
<style lang="scss" scoped>
.active {
  float: left;
  margin-left: 10px;
  padding: 10px;
  background: #e6a23c;
  margin-bottom: 10px;
  border-radius: 4px;
  font-size: 14px;
}

.Classification {
  float: left;
  margin-left: 10px;
  padding: 10px;
  background: #f7f7f7;
  margin-bottom: 10px;
  border-radius: 4px;
  font-size: 14px;
}

.select-voucher-dialog {
  .el-form-item {
    margin-bottom: 0;
  }

  .el-input.el-input--small {
    width: 200px;
  }

  .main-container {
    margin-bottom: 20px;
  }
}
</style>
