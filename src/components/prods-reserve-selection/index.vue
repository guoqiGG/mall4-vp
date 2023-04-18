<template>
  <el-dialog
    :title="$t('product.select')"
    top="100px"
    :append-to-body="visible"
    :close-on-click-modal="false"
    :visible.sync="visible"
    width="60%"
  >
    <el-form @submit.native.prevent :inline="true" :model="dataForm" size="small" class="demo-form-inline">
      <el-form-item :label="$t('product.prodName')">
        <el-input
          v-model="prodName"
          :placeholder="$t('product.prodName')"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item :label="$t('coupon.categoryParent')">
        <el-cascader
          expand-trigger="hover"
          :options="categoryList"
          :props="categoryTreeProps"
          :clearable="true"
          v-model="selectedCategory"
          @change="handleChange"
        ></el-cascader>
      </el-form-item>
      <el-form-item>
        <div class="default-btn primary-btn" @click="searchProd">{{ $t("coupon.query") }}</div>
        <div class="default-btn" @click="clean">{{ $t("product.reset") }}</div>
      </el-form-item>
    </el-form>
    <div class="prods-select-body">
      <el-table
        ref="prodTable"
        :data="dataList"
        header-cell-class-name="table-header"
        row-class-name="table-row"
        height="500"
        v-loading="dataListLoading"
        :row-key="getRowKeys"
        @selection-change="selectChangeHandle"
        @select="selectHandle"
        @select-all="selectAll"
        style="width: 100%"
      >
        <el-table-column
          v-if="isSingle"
          width="50"
          header-align="center"
          align="center"
        >
          <template slot-scope="scope">
            <div>
              <el-radio
                :label="scope.row.prodId"
                v-model="singleSelectProdId"
                @change.native="getSelectProdRow(scope.row)"
                >&nbsp;</el-radio
              >
            </div>
          </template>
        </el-table-column>
        <el-table-column
          v-if="!isSingle"
          type="selection"
          header-align="center"
          align="center"
          width="50"
        ></el-table-column>
        <el-table-column
          prop="prodName"
          header-align="center"
          align="center"
          :label="$t('coupon.commodityName')"
        ></el-table-column>
        <el-table-column align="center" width="140" :label="$t('product.pic')">
          <template slot-scope="scope">
            <ImgShow :src="scope.row.pic" :imgStyle="{width:'100px',height:'100px'}" />
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <div class="pagination">
        <el-pagination
          @size-change="sizeChangeHandle"
          @current-change="currentChangeHandle"
          :current-page="pageIndex"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="pageSize"
          :total="totalPage"
          layout="total, sizes, prev, pager, next, jumper"
        ></el-pagination>
      </div>
    </div>
    <span slot="footer">
      <div class="default-btn" @click="visible = false">{{ $t("coupon.cancel") }}</div>
      <div class="default-btn primary-btn" @click="submitProds()">{{ $t("coupon.confirm") }}</div>
    </span>
  </el-dialog>
</template>

<script>
import { treeDataTranslate, idList } from '@/utils'
export default {
  data () {
    return {
      visible: false,
      dataForm: {
        product: ''
      },
      singleSelectProdId: 0,
      allData: [],
      selectProds: [],
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      prodName: null,
      categoryList: [],
      selectedCategory: [],
      categoryTreeProps: {
        value: 'categoryId',
        label: 'categoryName'
      },
      resourcesUrl: process.env.VUE_APP_RESOURCES_URL
    }
  },
  props: {
    isSingle: {
      default: false,
      type: Boolean
    },
    prodType: {
      default: null,
      type: Number
    },
    dataUrl: {
      default: '/platform/search/prod/page',
      type: String
    }
  },
  activated () {
    this.getDataList()
  },
  beforeDestroy () {
    this.$refs.prodTable.clearSelection()
  },
  methods: {
    // 获取数据列表
    init (selectProds) {
      this.singleSelectProdId = 0
      this.selectProds = selectProds
      this.visible = true
      this.dataListLoading = true
      if (this.selectProds) {
        this.selectProds.forEach(row => {
          if (row.prodId) this.dataListSelections.push(row)
        })
      }
      this.getDataList()
      this.getCategoryList()
    },
    getCategoryList () {
      this.$http({
        url: this.$http.adornUrl('/prod/category/listCategory'),
        method: 'get',
        params: this.$http.adornParams({
          maxGrade: 2,
          status: 1
        })
      }).then(({ data }) => {
        this.categoryList = treeDataTranslate(data, 'categoryId', 'parentId')
      })
    },
    getDataList () {
      this.$http({
        url: this.$http.adornUrl(this.dataUrl),
        method: 'get',
        params: this.$http.adornParams(
          Object.assign(
            {
              current: this.pageIndex,
              size: this.pageSize,
              prodName: this.prodName ? this.prodName : null,
              keyword: this.prodName ? this.prodName : null,
              categoryId: this.categoryId ? this.categoryId : null,
              status: 1,
              prodType: this.prodType,
              isActive: 1 // 过滤掉活动商品
            }
            // {
            //   prodName: this.dataForm.prodName
            // }
          )
        )
      }).then(({ data }) => {
        this.dataList = data.records
        this.totalPage = data.total
        this.dataListLoading = false
        if (this.selectProds && this.selectProds.length) {
          this.$nextTick(() => {
            // this.selectProds.forEach(row => {
            //   let index = this.dataList.findIndex((prodItem) => prodItem.prodId === row.prodId)
            //   if (index >= 0) {
            //     this.$refs.prodTable.toggleRowSelection(this.dataList[index], true)
            //   }
            // })

            if (this.dataListSelections.length > 0) {
              // console.log('dataList:', this.dataList)
              this.dataList.map(element => {
                if (this.dataListSelections.map(item => item.prodId).indexOf(element.prodId) !== -1) {
                  console.log('element:', element, 'dataListSelections', this.dataListSelections)
                  this.$refs.prodTable.toggleRowSelection(element, true)
                }
              })
            }
          })
          // this.$nextTick(() => {
          //   this.dataListSelections.forEach(row => {
          //     this.$refs.prodTable.toggleRowSelection(row, true)
          //   })
          //   this.$forceUpdate()
          // })
        }
      })
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    },
    // 单选商品事件
    getSelectProdRow (row) {
      this.dataListSelections = [row]
    },
    // 全选
    selectAll (e) {
      if (e.length > 0) {
        this.dataListSelections = this.dataListSelections.filter(value => this.dataList.map(val => val.prodId).indexOf(value.prodId) === -1)
        this.dataList.map(value => {
          this.dataListSelections.push(value)
        })
      } else {
        this.dataListSelections = this.dataListSelections.filter(value => this.dataList.map(val => val.prodId).indexOf(value.prodId) === -1)
      }
      console.log(this.dataListSelections)
    },
    // 手动勾选数据行的 Checkbox 时触发的事
    selectHandle (selection, row) {
      console.log(selection, row)
        // 判断是否选中
      let selectionStatu = selection.map(element => element.prodId).indexOf(row.prodId) !== -1
      console.log(selection.map(element => element.prodId), row.prodId, selectionStatu)
      if (selectionStatu && selection.length > 0 && selection) {
          // 选中添加
        console.log('add:' + row.prodId)
        this.dataListSelections.push(row)
      } else {
          // 未选中删除
        this.dataListSelections = this.dataListSelections.filter(element => element.prodId !== row.prodId)
        console.log('del:' + row.prodId)
      }
      console.log(this.dataListSelections)
    },
    // 多选点击事件
    selectChangeHandle (selection) {
      // this.dataListSelections = selection
      // console.log(selection)
      // let dataListId = this.dataList.map(element => element.prodId)
      // console.log('dataListId:', dataListId)
      // // 去重
      // this.dataListSelections = this.dataListSelections.filter(element => dataListId.indexOf(element.prodId) === -1)
      // console.log('dataListSelections:', this.dataListSelections)
      // if (selection.length > 0) {
      //   selection.map(element => {
      //     if (element) this.dataListSelections.push(element)
      //   })
      // }
      // this.selectProds = this.dataListSelections
      // console.log('dataListSelections:', this.dataListSelections)
    },
    // isExist (dataListSelections, spu) {
    //   for (const item of dataListSelections) {
    //     if (item.prodId === spu.prodId) {
    //       return true
    //     }
    //   }
    //   return false
    // },
    getRowKeys (row) {
      return row.prodId
    },
    /**
     * 获取分类id
     */
    handleChange (val) {
      this.categoryId = val[val.length - 1]
    },
    /**
     * 根据条件搜索商品
     */
    searchProd () {
      this.pageIndex = 1
      this.getDataList()
    },
    /**
     * 清空搜索条件
     */
    clean () {
      this.prodName = ''
      this.categoryId = null
      this.selectedCategory = idList(this.categoryList, this.categoryId, 'categoryId', 'children').reverse()
    },
    // 确定事件
    submitProds () {
      // 商品单选情况
      if (this.isSingle) {
        this.dataListSelections.length && this.$emit('refreshSelectProds', this.dataListSelections[0])
        // 商品多选情况
      } else {
        let prods = []
        console.log(this.dataListSelections)
        this.dataListSelections.forEach(item => {
          let prodIndex = prods.findIndex((prod) => prod.prodId === item.prodId)
          if (prodIndex === -1) {
            prods.push(
              {
                prodId: item.prodId,
                prodName: item.prodName,
                pic: item.pic.includes('http') ? item.pic : this.resourcesUrl + item.pic,
                prodType: item.prodType,
                oriPrice: item.oriPrice,
                price: item.price,
                brief: item.brief,
                activityPrice: item.activityPrice
              }
            )
            //! item.pic.includes('http')
          }
        })
        this.$emit('refreshSelectProds', prods)
      }
      this.dataListSelections = []
      this.visible = false
    }
  }
}
</script>

<style lang="scss" scoped>
.pagination {
  margin-top: 15px;
}
</style>

