<template>
  <div class="mod-distribution-user">
    <!-- 搜索栏 -->
    <div class="search-bar">
      <el-form @submit.native.prevent :inline="true" class="search-form" ref="searchForm" :model="searchForm"
        size="small">
        <div class="input-row">
          <el-form-item label="团长手机号:">
            <el-input type="text" v-model="searchForm.userMobile" placeholder="团长手机号"></el-input>
          </el-form-item>
          <el-form-item :label="$t('distributionMsg.inviteesPhoneNumber') + ':'">
            <el-input type="text" v-model="searchForm.parentUserMobile"
              :placeholder="$t('distributionMsg.inviteesPhoneNumber')"></el-input>
          </el-form-item>
          <!--          状态-->
          <el-form-item :label="$t('distribUserWallet.status') + ':'">
            <el-select v-model="searchForm.state" :placeholder="$t('distribUserWallet.status')" clearable>
              <el-option :label="$t('distribUserWallet.perBan')" :value="-1"></el-option>
              <el-option :label="$t('publics.normal')" :value="1"></el-option>
              <el-option :label="$t('distribUserWallet.ban')" :value="2"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <div class="default-btn primary-btn" @click="searchChange(true)">{{ $t('crud.searchBtn') }}</div>
            <div class="default-btn" @click="resetSearchForm('searchForm')">{{ $t('product.reset') }}</div>
            <div class="default-btn" @click="getSoldExcel()">导出</div>
          </el-form-item>
        </div>
      </el-form>
    </div>

    <div class="main-container">
      <!-- 表格 -->
      <div class="table-con distribution-user-table">
        <el-table ref="distributionUserTable" :data="dataList" header-cell-class-name="table-header"
          row-class-name="table-row-low" style="width: 100%" @sort-change="sortChange">
          <!-- 昵称 -->
          <el-table-column fixed="left" prop="nickName" label="姓名" width="100">
            <template slot-scope="scope">
              {{ scope.row.realName }}
            </template>
          </el-table-column>
          <!-- 团长手机号 -->
          <el-table-column width="120" prop="userMobile" label="手机号">
            <template slot-scope="scope">
              {{ scope.row.userMobile }}
            </template>
          </el-table-column>
          <el-table-column width="120" label="等级">
            <template slot-scope="scope">
              {{ scope.row.distributionInfo ? scope.row.distributionInfo.name : '无' }}
            </template>
          </el-table-column>

          <el-table-column label="门店" width="200">
            <template slot-scope="scope">
              {{ scope.row.station?scope.row.station.stationName:'-' }}
            </template>
          </el-table-column>
          <el-table-column label="门店地址" width="200">
            <template slot-scope="scope">
              {{ scope.row.station?(scope.row.station.province + scope.row.station.city + scope.row.station.area + scope.row.station.addr):'-' }}
            </template>
          </el-table-column>
          <!-- 邀请人手机号 -->
          <!-- <el-table-column width="150" prop="parentUserMobile" :label="$t('distributionMsg.inviteesPhoneNumber')">
            <template slot-scope="scope">
              <span v-if="!scope.row.parentDistributionUser">无</span>
              <span v-else>{{ scope.row.parentDistributionUser.userMobile }}</span>
            </template>
          </el-table-column> -->
          <!-- 邀请人 -->
          <!-- <el-table-column prop="parentName" :label="$t('distributionMsg.invitees')">
            <template slot-scope="scope">
              <span v-if="!scope.row.parentDistributionUser">无</span>
              <span v-else>{{ scope.row.parentDistributionUser.nickName }}</span>
            </template>
          </el-table-column> -->
          <!-- 等级 -->
          <!-- <el-table-column
            prop="level"
            :label="$t('user.level')"
          >
            <template slot-scope="scope">
              {{ scope.row.levelName }}
            </template>
          </el-table-column> -->
          <!-- 加入时间 -->
          <el-table-column prop="bindTime" width="200" :label="$t('distributionMsg.joiningTime')" sortable="custom">
            <template slot-scope="scope">
              {{ scope.row.bindTime }}
            </template>
          </el-table-column>
          <!-- 累计客户 -->
          <el-table-column prop="boundCustomers" :label="$t('distributionMsg.cumulativeCustomers')" sortable="custom">
            <template slot-scope="scope">
              {{ scope.row.distributionUserAchievementDataDto.boundCustomers }}
            </template>
          </el-table-column>
          <!-- 累计邀请 -->
          <el-table-column prop="invitedVeeker" :label="$t('distributionMsg.cumulativeInvitation')" sortable="custom">
            <template slot-scope="scope">
              {{ scope.row.distributionUserAchievementDataDto.invitedVeeker }}
            </template>
          </el-table-column>
          <!-- 累计收益 -->
          <el-table-column prop="addupAmount" :label="$t('distributionMsg.cumulativeIncome')" sortable="custom">
            <template slot-scope="scope">
              {{ scope.row.distributionUserAchievementDataDto.distributionUserWallet.addupAmount }}
            </template>
          </el-table-column>
          <!-- 状态 -->
          <el-table-column prop="state" :label="$t('product.status')">
            <template slot-scope="scope">
              <div class="tag-text" v-if="scope.row.state === -1">{{ $t("distribUserWallet.perBan") }}</div>
              <div class="tag-text" v-if="scope.row.state === 0">{{ $t("product.pendingReview") }}</div>
              <div class="tag-text" v-if="scope.row.state === 1">{{ $t("publics.normal") }}</div>
              <div class="tag-text" v-if="scope.row.state === 2">{{ $t("distribUserWallet.ban") }}</div>
              <div class="tag-text" v-if="scope.row.state === 3">{{ $t("publics.unPassed") }}</div>
            </template>
          </el-table-column>
          <!-- 操作 -->
          <el-table-column fixed="right" align="center" :label="$t('publics.operating')" width="270">
            <template slot-scope="scope">
              <div class="text-btn-con">
                <div v-if="isAuth('distribution:distributionUserLevel:update')" class="default-btn text-btn"
                  @click="distributionLevelUpdateHandle(scope.row)">
                  修改等级
                </div>
                <div class="default-btn text-btn" @click="distributionReplaceUpdateHandle(scope.row)">
                  团长更换
                </div>
                <div v-if="isAuth('distribution:distributionUser:info')" class="default-btn text-btn"
                  @click="info(scope.row)">
                  {{ $t('seckill.edit') }}
                </div>
                <div v-if="isAuth('distribution:distributionUser:update') && scope.row.state !== -1"
                  class="default-btn text-btn" @click="addOrUpdateHandle(scope.row)">
                  {{ $t('distributionMsg.updateStatus') }}
                </div>
                <div>

                </div>
              </div>
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


    <!-- 弹窗, 新增 / 修改  状态-->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="refreshChange"></add-or-update>

    <!-- 弹窗, 新增 / 修改 信息 -->
    <InfoUpdate v-if="infoVisible" ref="info" @refreshDataList="refreshChange"></InfoUpdate>

    <!-- 修改团长等级 -->
    <DistributionLevelUpdate v-if="distributionLevelVisible" ref="distributionLevel" @refreshDataList="refreshChange">
    </DistributionLevelUpdate>
    <!-- 团长更换 -->
    <DistributionReplaceUpdate v-if="distributionReplaceVisible" ref="distributionReplace"
      @refreshDataList="refreshChange">
    </DistributionReplaceUpdate>
  </div>
</template>
<script>
import AddOrUpdate from './distribution-user-update'
import InfoUpdate from './distribution-user-info-update'
import DistributionLevelUpdate from './distribution-level-update'
import DistributionReplaceUpdate from './distribution-replace-update'
export default {
  data() {
    return {
      theData: null, // 保存上次点击查询的请求条件
      theParams: null, // 保存上次点击查询的请求条件

      dataForm: {
        // 团长手机号
        mobile: null,
        // 邀请人手机号
        parentMobile: null,
        // 加入时间区间
        dateRange: []
      },
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      searchForm: {},
      dataList: [],
      dataListLoading: false,
      addOrUpdateVisible: false,
      distributionLevelVisible: false,
      distributionReplaceVisible: false,
      infoVisible: false,
      sortParam: 1, // 0无 1加入时间 2累计客户 3累计邀请 4累计收益
      sortType: 2 // 0无 1 正序 2倒序
    }
  },
  components: {
    AddOrUpdate,
    InfoUpdate,
    DistributionLevelUpdate,
    DistributionReplaceUpdate
  },
  created() {
    this.getDataList()
  },
  methods: {
    getSoldExcel() {
      this.$http({
        url: this.$http.adornUrl('/platform/order/distribution/soldExcel'),
        method: 'get',
        params: this.$http.adornParams(),
        responseType: 'blob' // 解决文件下载乱码问题
      }).then(({ data }) => {
        var blob = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8' })
        const fileName = '团长信息整理'
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
      })
    },
    // 新增 / 修改
    addOrUpdateHandle(data) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(data)
      })
    },
    // 修改团长等级
    distributionLevelUpdateHandle(data) {
      this.distributionLevelVisible = true
      this.$nextTick(() => {
        this.$refs.distributionLevel.init(data)
      })
    },
    // 团长更换
    distributionReplaceUpdateHandle(data) {
      this.distributionReplaceVisible = true
      this.$nextTick(() => {
        this.$refs.distributionReplace.init(data)
      })
    },
    // 新增 / 修改
    info(data) {
      let aa = data
      this.infoVisible = true
      this.$nextTick(() => {
        this.$refs.info.init(aa)
      })
    },
    // 点击查询
    searchChange(newData = false) {
      this.page.currentPage = 1
      this.getDataList(this.page, newData)
    },
    // 刷新回调
    refreshChange() {
      this.page.currentPage = 1
      this.getDataList(this.page)
    },
    // 获取数据列表
    getDataList(page, newData = false) {
      if (newData || !this.theData) {
        this.theParams = JSON.parse(JSON.stringify(this.searchForm))
        this.theData = {
          current: page ? page.currentPage : this.page.currentPage,
          size: page ? page.pageSize : this.page.pageSize,
          sortParam: this.sortParam,
          sortType: this.sortType
        }
      } else {
        this.theData.current = page ? page.currentPage : this.page.currentPage
        this.theData.size = page ? page.pageSize : this.page.pageSize
      }
      this.$http({
        url: this.$http.adornUrl('/distribution/distributionUser/page'),
        method: 'get',
        params: this.$http.adornParams(Object.assign(this.theData, this.theParams))
      }).then(({ data }) => {
        this.page.total = data.total
        this.dataList = data.records
        this.dataListLoading = false

        // 末尾页数据为空重新请求
        if (!this.dataList.length && this.page.currentPage > 1) {
          this.page.currentPage = 1
          this.getDataList()
        }
      })
    },
    sortChange(data) {
      // 排序字段 0无 1加入时间 2累计客户 3累计邀请 4累计收益
      switch (data.prop) {
        case 'bindTime': this.sortParam = 1
          break
        case 'boundCustomers': this.sortParam = 2
          break
        case 'invitedVeeker': this.sortParam = 3
          break
        case 'addupAmount': this.sortParam = 4
          break
        default: this.sortParam = 1
          break
      }
      // 排序类型 0无 1 正序 2倒序
      switch (data.order) {
        case 'descending': this.sortType = 2
          break
        case 'ascending': this.sortType = 1
          break
        default: this.sortType = 2
          break
      }
      this.getDataList(this.page, true)
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
    }
  }
}
</script>

<style lang="scss" scoped></style>
