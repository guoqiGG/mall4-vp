<template>
  <div class="mod--distributionBasicSet">
    <div class="operation-bar">
      <div v-if="isAuth('platform:distributionLevel:save')" class="default-btn primary-btn" @click="addOrUpdateHandle(0)">
        {{ $t('crud.addBtn') }}</div>
    </div>
    <div class="main-container">
      <div class="table-con prod-table">
        <el-table ref="prodListTable" :data="dataList" header-cell-class-name="table-header" row-class-name="table-row"
          style="width: 100%">
          <el-table-column align="center" prop="name" label="等级名称" width="100" />
          <el-table-column align="center" prop="gradeRatio" label="团长提成比例" width="auto" />
          <el-table-column fixed="right" align="center" :label="$t('publics.operating')" width="230">
            <template slot-scope="scope">
              <div class="text-btn-con">
                <div class="default-btn text-btn" v-if="isAuth('platform:distributionLevel:update')" type="text"
                  @click="addOrUpdateHandle(scope.row.id)">修改</div>
                <div class="default-btn text-btn"
                  v-if="isAuth('platform:distributionLevel:delete')"
                  @click="deleteHandle(scope.row.id)">删除</div>
              </div>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </div>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="refreshChange"></add-or-update>
  </div>
</template>
<script>
import AddOrUpdate from './distribution-level-add-or-update.vue'
export default {
  data() {
    return {
      dataList: [],
      dataListLoading: false,
      addOrUpdateVisible: false
    }
  },
  components: {
    AddOrUpdate
  },
  created() {
    this.getDataList()
  },
  mounted() {
  },
  methods: {
    getDataList() {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/platform/distributionConfig/user/info'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({ data }) => {
        this.dataList = data
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
    deleteHandle(id) {
      this.$confirm(this.$t('seckill.isDeleOper'), this.$i18n.t('text.tips'), {
        confirmButtonText: this.$i18n.t('coupon.confirm'),
        cancelButtonText: this.$i18n.t('coupon.cancel'),
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/platform/distributionConfig/update/user/info'),
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
    /**
     * 刷新回调
     */
    refreshChange() {
      this.getDataList(this.page)
    },
    searchChange() {
      this.getDataList(this.page)
    }
  }
}
</script>
<style lang="scss"></style>
