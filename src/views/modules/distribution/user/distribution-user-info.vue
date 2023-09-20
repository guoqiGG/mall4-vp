<template>
  <div class="mod-user-info">
    <el-dialog :title="$t('distributionMsg.titleTip1')" :close-on-click-modal="false" :visible.sync="visible"
      @before-close="close" width="650px">
      <el-form @submit.native.prevent :model="dataForm" :rules="dataRule" ref="dataForm"
        @keyup.enter.native="dataFormSubmit()" :label-width="this.$i18n.t('language') === 'language' ? '180px' : '100px'">
        <el-form-item :label="$t('distributionMsg.distriTelPhone')" prop="userMobile">
          <el-input size="small" v-model="dataForm.userMobile"></el-input>
        </el-form-item>

        <!-- <el-form-item :label="$t('distribution.idCardNo')" prop="identityCardNumber">
          <el-input size="small" v-model="dataForm.identityCardNumber"></el-input>
        </el-form-item> -->

        <el-form-item :label="$t('distribution.realName')" prop="realName">
          <el-input size="small" v-model="dataForm.realName"></el-input>
        </el-form-item>
        <!-- <el-form-item label="个人信息">
          <div class="pic-con" v-if="infoList.length">
            <div class="pic-item" v-for="(pic, index) in infoList" :key="index">
              <el-image :src="pic.srcLink" :preview-src-list="srcList">
              </el-image>
              <span>{{ pic.title }}</span>
            </div>
          </div>
        </el-form-item> -->
        <el-form-item label="所在地区" prop="currentArea">
          <el-cascader ref="cascaderAddr" v-model="currentArea" :options="addrList" :props="addrListTreeProps"
            :placeholder="this.$i18n.t('shopProcess.addrInputTips')" class="addr-select" @change="selectAreaChange" />
        </el-form-item>

        <el-form-item label="详细地址" prop="address">
          <el-input v-model="address" :placeholder="this.$i18n.t('shopProcess.detailAddrInputTips')" maxlength="50" @blur="
            address =
            address ?
              removeHeadAndTailSpaces(address) :
              address; shopAddressChange()
            ">
            <el-button slot="append" icon="el-icon-location" @click="mapLocation">{{ $t("platform.location")
            }}</el-button>
          </el-input>
        </el-form-item>
        <!-- 地图 -->
        <el-form-item>
          <baidu-map ref="baiduMp" class="map" :scroll-wheel-zoom="false" :center="center" @moving="syncCenterAndZoom"
            @moveend="syncCenterAndZoom" @zoomend="syncCenterAndZoom" @ready="handler" :zoom="zoom">
            <bm-navigation anchor="BMAP_ANCHOR_TOP_RIGHT"></bm-navigation>
            <bm-marker :position="center" :dragging="false" @click="infoWindowOpen">
              <bm-info-window :show="show" @close="infoWindowClose" @open="infoWindowOpen">{{ $t("shop.storeLocation")
              }}</bm-info-window>
            </bm-marker>
          </baidu-map>
        </el-form-item>

        <!--
        <el-form-item :label="$t('distributionMsg.idCardFront')">
          <el-image
            v-if="dataForm.identityCardPicFront"
            style="width: 100px; height: 100px"
            :src="dataForm.identityCardPicFront"
            :preview-src-list="srcList">
          </el-image>
          <span v-else>无</span>
        </el-form-item>

        <el-form-item :label="$t('distributionMsg.idCardBack')">
          <el-image
            v-if="dataForm.identityCardPicBack"
            style="width: 100px; height: 100px"
            :src="dataForm.identityCardPicBack"
            :preview-src-list="srcList">
          </el-image>
          <span v-else>无</span>
        </el-form-item>

        <el-form-item :label="$t('distributionMsg.holdIdCard')">
          <el-image
            v-if="dataForm.identityCardPicHold"
            style="width: 100px; height: 100px"
            :src="dataForm.identityCardPicHold"
            :preview-src-list="srcList">
          </el-image>
          <span v-else>无</span>
        </el-form-item> -->

      </el-form>
      <span slot="footer" class="dialog-footer">
        <div class="default-btn" @click="close()">{{ $t('seckill.close') }}</div>
        <div class="default-btn primary-btn" @click="confirmSubmit()">{{ $t('remindPop.confirm') }}</div>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { validEmail, isCreditCode, validNoEmptySpace, removeHeadAndTailSpaces } from '@/utils/validate'

export default {
  data() {
    var validateArea = (rule, value, callback) => {
      if (!this.currentArea) {
        callback(new Error('所在地区不能为空'))
      } else {
        callback()
      }
    }
    var validateAddr = (rule, value, callback) => {
      if (!this.address) {
        callback(new Error('详细地址不能为空'))
      } else {
        callback()
      }
    }
    return {
      removeHeadAndTailSpaces: removeHeadAndTailSpaces,
      dataRule: {
        realName: [
          { required: true, message: '姓名不能为空', trigger: 'blur' }
        ],
        userMobile: [
          { required: true, message: '手机号不能为空', trigger: 'blur' }
        ],
        identityCardNumber: [
          { required: true, message: '身份证号不能为空', trigger: 'blur' }
        ],
        currentArea: [
          { required: true, validator: validateArea, trigger: 'blur' }
        ],
        address: [
          { required: true, validator: validateAddr, trigger: 'blur' }
        ],
      },
      dataForm: {
        userMobile: '',
        realName: '',
        identityCardNumber: ''
      },
      addProdVisible: false,
      visible: false,
      srcList: [],
      infoList: [],
      addrList: [],
      addrListTreeProps: {
        value: 'areaId',
        label: 'areaName',
        children: 'areas'
      },
      baiduIndex: false,
      currentArea: [], // 所在地区
      address: '', // 详细地址
      zoom: 18,
      center: {
        lng: 113.327955,
        lat: 23.136783
      },
      center2: {
        lng: 113.327955,
        lat: 23.136783
      },
      // 定位
      show: false,
    }
  },
  components: {
  },
  methods: {
    init(data) {
      console.log(data)
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        this.dataForm = Object.assign(this.dataForm, data)

        this.currentArea = [
          this.dataForm.station.provinceId,
          this.dataForm.station.cityId,
          this.dataForm.station.areaId
        ]
        if (data.station.lat && data.station.lng) {
          if (this.baiduIndex) {
            this.center = {
              lng: Number.parseFloat(data.station.lng),
              lat: Number.parseFloat(data.station.lat)
            }
          } else {
            this.center2 = {
              lng: Number.parseFloat(data.station.lng),
              lat: Number.parseFloat(data.station.lat)
            }
          }
        }
        this.address = this.dataForm.station.addr
        let infoList = []
        if (this.dataForm.identityCardPicFront) {
          infoList.push({
            title: this.$i18n.t('distributionMsg.idCardFront'),
            srcLink: this.dataForm.identityCardPicFront
          })
        }
        if (this.dataForm.identityCardPicBack) {
          infoList.push({
            title: this.$i18n.t('distributionMsg.idCardBack'),
            srcLink: this.dataForm.identityCardPicBack
          })
        }
        if (this.dataForm.identityCardPicHold) {
          infoList.push({
            title: this.$i18n.t('distributionMsg.holdIdCard'),
            srcLink: this.dataForm.identityCardPicHold
          })
        }
        this.infoList = infoList
        if (infoList.length) {
          for (let i = 0; i < infoList.length; i++) {
            const element = infoList[i]
            this.srcList.push(element.srcLink)
          }
        }
      })
      // 获取地址列表
      this.getAddrList()
      // 初始化地图数据
      this.setCenter()
    },
    handler({ BMap, map }) {
      this.baiduIndex = true
      this.center = { ...this.center2 }
    },
    /**
    * 省市区
    */
    getAddrList() {
      // addrApi.listAreaOfEnable({}).then(data => {
      //   this.addrList = treeDataTranslate(data, 'areaId', 'parentId')
      // })
      this.$http({
        url: this.$http.adornUrl(`/admin/area/listAreaOfEnable`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({ data }) => {
        this.addrList = data
      })
    },
    shopAddressChange() {
      this.mapLocation(true)
      this.dataForm.station.addr = this.address
      console.log(this.dataForm.station.addr,this.address)
    },
    selectAreaChange(value) {
      if (value.length > 0) {
        const labels = this.$refs['cascaderAddr'].getCheckedNodes()[0].pathLabels
        this.dataForm.station.province = labels[0]
        this.dataForm.station.provinceId = value[0]
        this.dataForm.station.city = labels[1]
        this.dataForm.station.cityId = value[1]
        this.dataForm.station.area = labels[2]
        this.dataForm.station.areaId = value[2]
        this.mapLocation(true)
      }
    },
    close() {
      this.visible = false
      this.$refs['dataForm'].resetFields()
    },
    confirmSubmit() {
      this.dataFormSubmit()
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(`/distribution/distributionUser/update/status`),
            method: 'post',
            data: this.$http.adornData({
              distributionUserId: this.dataForm.distributionUserId,
              userMobile: this.dataForm.userMobile,
              realName: this.dataForm.realName,
              province: this.dataForm.station.province,
              provinceId: this.dataForm.station.provinceId,
              city: this.dataForm.station.city,
              cityId: this.dataForm.station.cityId,
              area: this.dataForm.station.area,
              areaId: this.dataForm.station.areaId,
              addr: this.dataForm.station.addr,
              lat: this.dataForm.station.lat,
              lng: this.dataForm.station.lng
            })

          }).then(({ data }) => {
            this.$message({
              message: this.$i18n.t('publics.operation'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.visible = false
                this.$emit('refreshDataList')
                this.currentArea = []
                this.address = ''
                this.$refs['dataForm'].resetFields()
              }
            })
          })
        }
      })
    },
    /**
   * 初始化地图数据
   */
    setCenter() {
      this.center = null
      this.center = {
        lng: 113.327955,
        lat: 23.136783
      }
      this.zoom = 18
    },
    /**
     * 获取地图移动后回调的位置参数
     */
    syncCenterAndZoom(e) {
      this.setCenter()
      const { lng, lat } = e.target.getCenter()
      this.center.lng = lng
      this.center.lat = lat
      this.zoom = e.target.getZoom()
      this.dataForm.station.lng = lng
      this.dataForm.station.lat = lat
    },
    /**
     * 关闭地图说明标签
     */
    infoWindowClose() {
      this.show = false
    },
    /**
     * 打开地图说明标签
     */
    infoWindowOpen() {
      this.show = true
    },
    // 定位地图
    mapLocation(isTrue) {
      let area = ''
      const that = this
      if (isTrue === true) {
        // 选择省市区时，定位地图
        if (this.dataForm.station.provinceId) {
          area = this.dataForm.station.province
          this.zoom = 6
          if (this.dataForm.station.cityId) {
            area = area + this.dataForm.station.city
            this.zoom = 10
            if (this.dataForm.station.areaId) {
              area = area + this.dataForm.station.area
              this.zoom = 14
              if (this.dataForm.station.addr !== null) {
                // 详细地址
                area = this.dataForm.station.province + this.dataForm.station.city + this.dataForm.station.area + this.dataForm.station.addr
                this.zoom = 18
              }
            }
          }
        }
      } else {
        if (!this.dataForm.station.provinceId || !this.dataForm.station.cityId || !this.dataForm.station.areaId) {
          this.dataForm.station.addr = null
          this.$message({
            message: this.$i18n.t('admin.selePCD'),
            type: 'error',
            duration: 1000
          })
          return
        }
        let point = new BMap.Point(this.center.lng, this.center.lat)
        let gc = new BMap.Geocoder()
        gc.getLocation(point, function (rs) {
          if (rs) {
            let addrDetail = that.formatAddrDetail(rs.address)
            if (!addrDetail) {
              if (rs.surroundingPois && rs.surroundingPois.length > 0) {
                addrDetail = that.formatAddrDetail(rs.surroundingPois[0].address)
              }
            }
            (that.center = rs.point)
            if (addrDetail) {
              that.dataForm.station.addr = addrDetail
              that.address = addrDetail
              that.center = rs.point
              that.dataForm.station.lng = rs.point.lng
              that.dataForm.station.lat = rs.point.lat
            }
          }
        })
        this.zoom = 18
      }

      this.center = area || this.center
      this.isEditAddr = true
    },
    // 格式化详细地址显示
    formatAddrDetail(addrInfo) {
      const rex = new RegExp(`[${this.dataForm.station.province}|${this.dataForm.station.city}|${this.dataForm.station.area}]`, 'g')
      const result = addrInfo.replace(rex, '')
      return result
    }
  }
}
</script>

<style lang="scss" scoped>
.mod-user-info ::v-deep .el-dialog__body {
  padding-bottom: 0;
}

.pic-con {
  display: flex;

  .pic-item {
    display: flex;
    flex-direction: column;
    margin-right: 20px;

    .el-image {
      width: 80px;
      height: 80px;
    }

    span {
      font-size: 14px;
      color: #999;
    }
  }
}
</style>

<style lang="scss" scoped>
.mod-user-info ::v-deep .el-dialog__body {
  padding-bottom: 0;
}

.pic-con {
  display: flex;

  .pic-item {
    display: flex;
    flex-direction: column;
    margin-right: 20px;

    .el-image {
      width: 80px;
      height: 80px;
    }

    span {
      font-size: 14px;
      color: #999;
    }
  }
}
</style>
