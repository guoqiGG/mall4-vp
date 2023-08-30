<template>
  <div class="mod-distribution-user">
    <!-- 搜索栏 -->
    <div class="search-bar">
      <el-form @submit.native.prevent :inline="true" class="search-form" ref="searchForm" :model="searchForm"
        size="small">
        <div class="input-row">
          <el-form-item :label="$t('distributionBind.customer') + ':'">
            <el-input type="text" v-model="searchForm.nickName" :placeholder="$t('distributionBind.customer')"></el-input>
          </el-form-item>
          <el-form-item :label="$t('distributionProdLog.distributor') + ':'">
            <el-input type="text" v-model="searchForm.parentNickName"
              :placeholder="$t('distributionProdLog.distributor')"></el-input>
          </el-form-item>
          <el-form-item :label="$t('distributionBind.userMobile') + ':'">
            <el-input type="text" v-model="searchForm.cUserMobile"
              :placeholder="$t('distributionBind.userMobile')"></el-input>
          </el-form-item>
          <el-form-item :label="$t('distributionProdLog.userMobile') + ':'">
            <el-input type="text" v-model="searchForm.dUserMobile"
              :placeholder="$t('distributionProdLog.userMobile')"></el-input>
          </el-form-item>
          <el-form-item prop="state" :label="$t('distributionBind.currentRelationship') + ':'">
            <el-select v-model="searchForm.state" :placeholder="$t('distributionBind.currentRelationship')" clearable>
              <el-option :label="$t('seckill.loseEffectiveness')" :value="-1"></el-option>
              <el-option :label="$t('distributionBind.binding')" :value="1"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <div class="default-btn primary-btn" @click="searchChange(true)">{{ $t('crud.searchBtn') }}</div>
            <div class="default-btn" @click="resetSearchForm('searchForm')">{{ $t('product.reset') }}</div>
          </el-form-item>
        </div>
      </el-form>
    </div>

    <div class="main-container">
      <!-- 表格 -->
      <div class="table-con distribution-bind-table">
        <el-table ref="distributionBindTable" :data="dataList" header-cell-class-name="table-header"
          row-class-name="table-row-low" style="width: 100%" @sort-change="sortChange">
          <!-- 序号 -->
          <el-table-column type="index" width="100" :label="$t('formData.serialNumber')">
          </el-table-column>
          <!-- 客户 -->
          <el-table-column prop="nickName" :label="$t('distributionBind.customer')">
            <template slot-scope="scope">
              {{ scope.row.user.nickName }}
            </template>
          </el-table-column>
          <!--          客户号码-->
          <el-table-column prop="userMobile" :label="$t('distributionBind.userMobile')">
            <template slot-scope="scope">
              {{ scope.row.user.userMobile }}
            </template>

          </el-table-column>
          <!-- 团长等级 -->
          <el-table-column label="团长等级">
            <template slot-scope="scope">
              {{ scope.row.distributionInfo ? scope.row.distributionInfo.name : '无' }}
            </template>

          </el-table-column>
          <!-- 团长 -->
          <el-table-column prop="parentNickName" :label="$t('distributionProdLog.distributor')">
            <template slot-scope="scope">
              <span v-if="scope.row.distributionUser">{{ scope.row.distributionUser.nickName }}</span>
            </template>
          </el-table-column>
          <!--团长号码-->
          <el-table-column prop="userMobile" :label="$t('distributionProdLog.userMobile')">
            <template slot-scope="scope">
              {{ scope.row.distributionUser.userMobile }}
            </template>

          </el-table-column>

          <!-- 绑定时间 -->
          <el-table-column prop="bindTime" :label="$t('distribution.bindingTime')" sortable="custom">
            <template slot-scope="scope">
              {{ scope.row.bindTime }}
            </template>
          </el-table-column>
          <!-- 失效时间 -->
          <el-table-column prop="invalidTime" :label="$t('distribution.invalidTime')" sortable="custom">
            <template slot-scope="scope">
              <span v-if="!scope.row.invalidTime">无</span>
              <span v-else>{{ scope.row.invalidTime }}</span>
            </template>
          </el-table-column>
          <!-- 操作 -->
          <el-table-column prop="operation" label="操作">
            <template slot-scope="scope" v-if="scope.row.distributionUserId">
              <div class="default-btn text-btn" @click="showEditDialog(scope.row)">修改</div>
            </template>
          </el-table-column>
          <!-- 当前关系 -->
          <el-table-column align="center" prop="state" :label="$t('distributionBind.currentRelationship')" width="180">
            <template slot-scope="scope">
              <div class="tag-text" v-if="scope.row.state === -1">{{ $t("seckill.loseEffectiveness") }}</div>
              <div class="tag-text" v-if="scope.row.state === 1">{{ $t("distributionBind.binding") }}</div>
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
    <!-- 修改关系弹窗 -->
    <el-dialog :visible="editDialogVisible" title="修改关系" @close="closeEditDialog">
      <el-form label-width="80px" ref="editForm" :rules="rules" :model="editForm">
        <el-form-item label="客户">
          {{ editRecord.user.nickName }}
        </el-form-item>
        <el-form-item label="客户号码">
          {{ editRecord.user.userMobile }}
        </el-form-item>
        <el-form-item label="团长" prop="parentId">
          <el-select v-model="editForm.parentId" filterable remote reserve-keyword clearable placeholder="请输入团长手机号查询"
            :remote-method="remoteMethod" :loading="loading">
            <el-option v-for="item in parentOptions" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleSubmitEdit">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { debounce } from 'lodash'
export default {
  data() {
    return {
      rules: {
        parentId: [
          { required: true, message: '请选择团长', trigger: 'blur' }
        ]
      },
      theData: null, // 保存上次点击查询的请求条件
      dataForm: {
        userName: '',
        parentName: '',
        state: '',
        invalidReason: '',
        bindTime: [],
        invalidTime: [],
        cUserMobile: '',
        dUserMobile: ''
      },
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      searchForm: {},
      dataList: [],
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      editDialogVisible: false,
      parentOptions: [],
      editForm: {
        parentId: undefined
      },
      editRecord: {
        user: {}
      },
      loading: false
    }
  },
  components: {
  },
  created() {
    this.getDataList()
  },
  methods: {
    sortChange(column) {
      console.log(column)
      const { prop, order } = column
      const sortParam = {}
      // sort: 1 代表绑定时间，2代表失效时间
      switch (prop) {
        case 'bindTime':
          sortParam.sort = 1
          break
        case 'invalidTime':
          sortParam.sort = 2
          break
      }
      // orderBy: 0代表从小到大，1代表从大到小
      if (order) {
        sortParam.orderBy = order === 'ascending' ? 2 : 1
      } else {
        sortParam.orderBy = ''
      }
      this.sortParam = sortParam
      this.getDataList(this.page)
    },
    // 刷新回调
    refreshChange() {
      this.page.currentPage = 1
      this.getDataList(this.page)
    },
    // 点击查询
    searchChange(newData = false) {
      this.page.currentPage = 1
      this.getDataList(this.page, newData)
    },
    // 获取数据列表
    getDataList(page, newData = false) {
      this.dataListLoading = true
      if (newData || !this.theData) {
        this.theData = JSON.parse(JSON.stringify(this.searchForm))
      }
      this.$http({
        url: this.$http.adornUrl('/distribution/distributionUserBind/page'),
        method: 'get',
        params: this.$http.adornParams(Object.assign({
          current: page ? page.currentPage : this.page.currentPage,
          size: page ? page.pageSize : this.page.pageSize,
          ...this.sortParam
        }, this.theData))
      }).then(({ data }) => {
        this.page.total = data.total
        this.page.pageSize = data.size
        this.page.currentPage = data.current

        this.dataList = data.records.map(item => ({
          ...item,
          distributionUser: item.distributionUser || {}
        }))
        this.dataListLoading = false
      })
    },

    /**
     * 重置表单
     * @param {String} formName 表单名称
     */
    resetSearchForm(formName) {
      this.$refs[formName].resetFields()
      this.searchForm = {}
    },

    handleSizeChange(val) {
      this.page.pageSize = val
      this.getDataList(this.page)
    },
    handleCurrentChange(val) {
      this.page.currentPage = val
      this.getDataList(this.page)
    },
    // 展示弹窗
    showEditDialog(scope) {
      this.editDialogVisible = true
      this.editRecord = scope
    },
    // 关闭弹窗
    closeEditDialog() {
      this.editDialogVisible = false
      this.editRecord = { user: {} }
      this.editForm = { parentId: undefined }
    },
    // 团长模糊查询
    remoteMethod: debounce(function (value) {
      this.loading = true
      this.$http({
        url: this.$http.adornUrl('/distribution/distributionUser/disPageLike'),
        method: 'get',
        params: this.$http.adornParams(Object.assign({
          userMobile: value
        }, this.theParams))
      }).then(({ data }) => {
        this.parentOptions = data.map(item => ({
          ...item,
          label: `${item.nickName}(${item.userMobile})`,
          value: item.distributionUserId
        }))
      }).finally(() => {
        this.loading = false
      })
    }, 300),
    // 提交修改
    handleSubmitEdit() {
      this.$refs.editForm.validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl('/distribution/distributionUser/changeParentDistribution'),
            method: 'get',
            params: this.$http.adornParams(Object.assign({
              userId: this.editRecord.user.userId,
              parentId: this.editForm.parentId
            }, this.theParams))
          }).then(({ success }) => {
            if (success) {
              this.$message.success('修改成功')
              this.editDialogVisible = false
              this.getDataList(this.page)
            } else {
              this.$message.error('修改失败，请重试')
            }
          })
        } else {
          return false
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped></style>
