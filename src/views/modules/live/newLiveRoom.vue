<template>
  <div class="mod-live-room-add-or-update">
    <div class="new-page-title">
      <div class="line" />
      <div class="text">
        {{
          !dataForm.id ? '新增直播间' : '直播间信息'
        }}
      </div>
    </div>
    <el-form @submit.native.prevent
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      size="smalls"
      @keyup.enter.native="dataFormSubmit()"
      label-width="120px"
    >
      <el-form-item label="直播间类型" prop="type">
        <el-radio v-model="dataForm.type" :label="0">手机直播</el-radio>
<!--        <el-radio disabled v-model="dataForm.type" :label="1">{{$t("live.pushStream")}}</el-radio>-->
      </el-form-item>
      <el-form-item label="直播间名称" prop="name">
        <el-input
          size="small"
          v-model="dataForm.name "
          :disabled="dataForm.id !== 0 "
          maxlength="17"
          show-word-limit
        ></el-input>
      </el-form-item>
      <el-form-item label="主播昵称" prop="anchorName">
        <el-input
          size="small"
          v-model="dataForm.anchorName"
          maxlength="15"
          :disabled="dataForm.id !== 0 "
          show-word-limit
        ></el-input>
      </el-form-item>
      <el-form-item label="主播微信号" prop="anchorWechat">
        <el-input
          size="small"
          v-model="dataForm.anchorWechat"
          maxlength="20"
          :disabled="dataForm.id !== 0 "
          show-word-limit
        ></el-input>
      <p class="live-tips">每个直播间需要绑定一个用作核实主播身份，不会展示给观众。</p>
      </el-form-item>
      <!-- <el-form-item :label="直播的时间：" prop="dateRange">
        <el-date-picker
          v-model="dataForm.dateRange"
          type="datetimerange"
          :range-separator="this.$i18n.t('time.tip')"
          value-format="yyyy-MM-dd HH:mm:ss"
          :start-placeholder="this.$i18n.t('time.start')"
          :end-placeholder="this.$i18n.t('time.end')"
          :disabled="!!dataForm.seckillId"
        ></el-date-picker>
        <br />
        <span>开播时间需要在当前时间的10分钟后 并且开始时间不能在6个月后,开播时间和结束时间间隔不得短于30分钟，不得超过24小时</span>
      </el-form-item>-->
      <el-form-item label="直播的时间" class="liveTime">
        <div class="live-time-row">
          <el-form-item prop="startTime">
            <el-date-picker
              size="small"
              :disabled="dataForm.id !== 0 "
              v-model="dataForm.startTime"
              type="date"
              :placeholder="this.$i18n.t('live.chooseStartDate')"
              :picker-options="pickerOptions"
              value-format="yyyy-MM-dd"
              style="width:144px"
            ></el-date-picker>
            <el-time-select
              v-model="startTimeValue"
              :picker-options="{
                start: '00:00',
                step: '00:30',
                end: '23:30'
              }"
              size="small"
              style="width:101px"
              :disabled="dataForm.id !== 0"
              placeholder="生效时间">
            </el-time-select>
          </el-form-item>
          <div class="tip">至</div>
          <el-form-item prop="endTime">
            <el-date-picker
              size="small"
              v-model="dataForm.endTime"
              :disabled="dataForm.id !== 0 "
              type="date"
              placeholder="选择结束日期"
              :picker-options="pickerOptions"
              value-format="yyyy-MM-dd"
              style="width:144px"
            ></el-date-picker>
            <el-time-select
              v-model="endTimeValue"
              :picker-options="{
                start: '00:00',
                step: '00:30',
                end: '23:30'
              }"
              size="small"
              style="width:101px"
              :disabled="dataForm.id !== 0"
              placeholder="失效时间">
            </el-time-select>
          </el-form-item>
          <br />
        </div>
      </el-form-item>
      <el-form-item>
        <p class="live-tips" >开播时间需要在当前时间的15分钟后 并且开始时间不能在6个月后,开播时间和结束时间间隔不得短于30分钟，不得超过24小时</p>
      </el-form-item>
      <el-form-item label="直播背景图" prop="coverImg">
        <img-upload v-model="dataForm.coverImg" :maxSize="2" :disabled="dataForm.id !== 0 " />
        <p class="live-tips" >建议尺寸：1080像素 * 1920像素，图片大小不得超过2M</p>
      </el-form-item>
      <el-form-item label="主播分享图" prop="shareImg">
        <img-upload v-model="dataForm.shareImg" :maxSize="1" :disabled="dataForm.id !== 0 " />
        <p class="live-tips" >建议尺寸：800像素 * 640像素，图片大小不得超过1M。</p>
      </el-form-item>
      <el-form-item label="直播封面图" prop="feedsImg">
        <img-upload v-model="dataForm.feedsImg" :maxSize="0.1" :disabled="dataForm.id !== 0 " />
        <p class="live-tips" >建议尺寸：图片建议大小为 800像素 * 800像素，图片大小不得超过100KB。</p>
      </el-form-item>
      <el-form-item label="是否被收录" prop="isFeedsPublic">
        <el-radio
          v-model="dataForm.isFeedsPublic"
          :label="1"
          :disabled="dataForm.id !== 0 "
        >开启</el-radio>
        <el-radio
          v-model="dataForm.isFeedsPublic"
          :label="0"
          :disabled="dataForm.id !== 0 "
        >关闭</el-radio>
        <br />
        <p class="live-tips" >开启后本场直播将有可能被官方推荐,直播间创建完成后可以再修改。</p>
      </el-form-item>
      <el-form-item label="显示的方式" prop="screenType">
        <el-radio
          v-model="dataForm.screenType"
          :label="0"
          :disabled="dataForm.id !== 0 "
        >竖屏</el-radio>
        <el-radio v-model="dataForm.screenType" :label="1" disabled>横屏</el-radio>
        <!-- :disabled="dataForm.id !== 0 ||" -->
      </el-form-item>
      <el-form-item label="直播间功能" prop="closeLike">
        <el-checkbox v-model="closeLike" :disabled="dataForm.id !== 0 ">点赞</el-checkbox>
        <el-checkbox v-model="closeComment" :disabled="dataForm.id !== 0 ">评论</el-checkbox>
        <el-checkbox
          v-model="closeGoods"
          :disabled="dataForm.id !== 0 "
        >商品货架</el-checkbox>
        <br />
        <el-checkbox v-model="closeReplay" :disabled="dataForm.id !== 0 ">回放</el-checkbox>
        <el-checkbox v-model="closeShare" :disabled="dataForm.id !== 0 ">分享</el-checkbox>
        <el-checkbox v-model="closeKf" :disabled="dataForm.id !== 0 ">客服</el-checkbox>
      </el-form-item>
      <el-form-item>
        <div class="default-btn" @click="back()">取消</div>
          <div
            v-if="!dataForm.id"
            class="default-btn primary-btn"
            @click="dataFormSubmit()"
          >确定</div>
          <div
            v-if="dataForm.id"
            class="default-btn primary-btn"
            @click="back()"
          >确定</div>
        </el-form-item>
    </el-form>
  </div>
</template>

<script>
import ImgUpload from '@/components/img-upload'
import { getDateTimeRange, getParseTime } from '@/utils/datetime'
export default {
  name: 'new-live-room',
  data () {
    var validatorDateRange = (rule, value, callback) => {
      if (rule.field === 'startTime' && (!this.startTimeValue || !this.dataForm.startTime)) {
        callback(new Error(this.$i18n.t('publics.noNull')))
      }
      if (rule.field === 'endTime' && (!this.endTimeValue || !this.dataForm.endTime)) {
        callback(new Error(this.$i18n.t('publics.noNull')))
      }
      let startTime = this.dataForm.startTime + ' ' + this.startTimeValue + ':00'
      let endTime = this.dataForm.endTime + ' ' + this.endTimeValue + ':00'
      if (rule.field === 'startTime' && new Date() > Date.parse(startTime)) {
        callback(new Error(this.$i18n.t('groups.startTime')))
      }
      if (rule.field === 'startTime' && Date.parse(startTime) >= Date.parse(endTime)) {
        callback(new Error(this.$i18n.t('marketing.timeCanThanOrEqualTo')))
      }
      if (rule.field === 'endTime' && new Date() > Date.parse(endTime)) {
        callback(new Error(this.$i18n.t('groups.endTime')))
      }
      callback()
    }
    return {
      pickerOptions: {
        disabledDate (time) {
          return time.getTime() < (new Date(new Date(new Date().toLocaleDateString()).getTime()).getTime())
        }
      },
      visible: false,
      isSubmit: false,
      dataForm: {
        id: null,
        roomId: null,
        name: null,
        anchorName: null,
        anchorWechat: null,
        coverImg: null,
        shareImg: null,
        feedsImg: null,
        isFeedsPublic: 1,
        type: 0,
        screenType: 0,
        closeLike: true,
        closeGoods: true,
        closeComment: true,
        closeReplay: true,
        closeShare: false,
        closeKf: false,
        roomToolsVo: {
          closeLike: 0,
          closeGoods: 0,
          closeComment: 0,
          closeReplay: 0,
          closeShare: 0,
          closeKf: 0
        },
        dateRange: [],
        liveStatus: null,
        startTime: null,
        endTime: null,
        applyTime: null
      },
      closeLike: true,
      closeGoods: true,
      closeComment: true,
      closeReplay: true,
      closeShare: false,
      closeKf: false,
      dataRule: {
        name: [
          { required: true, message: '直播间名称不能为空', trigger: 'blur' }
        ],
        anchorName: [
          { required: true, message: '主播昵称不能为空', trigger: 'blur' }
        ],
        anchorWechat: [
          { required: true, message: '主播微信号不能为空', trigger: 'blur' }
        ],
        coverImg: [
          { required: true, message: '背景图不能为空', trigger: 'blur' }
        ],
        shareImg: [
          { required: true, message: '主播分享图不能为空', trigger: 'blur' }
        ],
        feedsImg: [
          { required: true, message: '直播频道封面不能为空', trigger: 'blur' }
        ],
        dateRange: [
          { required: true, message: '直播时间不能为空', trigger: 'blur' }
        ],
        startTime: [
          { required: true, message: this.$i18n.t('publics.noNull'), trigger: 'blur' },
          { required: true, validator: validatorDateRange, trigger: 'blur' }
        ],
        endTime: [
          { required: true, message: this.$i18n.t('publics.noNull'), trigger: 'blur' },
          { required: true, validator: validatorDateRange, trigger: 'blur' }
        ]
      },
      startTimeValue: '',
      endTimeValue: ''
    }
  },
  components: {
    ImgUpload
  },
  mounted () {
    const flag = sessionStorage.getItem('bbcLiveRoomData') !== 'undefined'
    const data = flag ? JSON.parse(sessionStorage.getItem('bbcLiveRoomData')) : null
    this.init(data)
    let title = !this.dataForm.id ? this.$t('live.addNewLiveRoom') : this.$t('live.viewLRoomInfo')
    this.$store.commit('common/replaceSelectMenu', title)
  },
  methods: {
    init (data) {
      console.log(data)
      if (data) {
        this.dataForm = data
        this.dataForm.id = data.id
      } else {
        this.dataForm.id = 0
      }
      this.closeLike = true
      this.closeGoods = true
      this.closeComment = true
      this.closeReplay = true
      this.closeShare = false
      this.closeKf = false
      this.visible = true
      this.isSubmit = false
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        let datetimeRange = getDateTimeRange()
        this.dataForm.startTime = datetimeRange.startTime
        this.dataForm.endTime = datetimeRange.endTime
        this.startTimeValue = datetimeRange.currentTime
        this.endTimeValue = datetimeRange.currentTime
        console.log(this.dataForm)
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl('/live/liveRoom/info/' + this.dataForm.id),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({ data }) => {
            this.dataForm = data
            this.startTimeValue = this.dataForm.startTime ? this.dataForm.startTime.substring(11, this.dataForm.startTime.length - 3) : ''
            this.endTimeValue = this.dataForm.endTime ? this.dataForm.endTime.substring(11, this.dataForm.endTime.length - 3) : ''
            this.dataForm.startTime = getParseTime(this.dataForm.startTime, '{y}-{m}-{d}')
            this.dataForm.endTime = getParseTime(this.dataForm.endTime, '{y}-{m}-{d}')
            if (data.roomToolsVo != null) {
              this.closeLike = data.roomToolsVo.closeLike === 0
              this.closeGoods = data.roomToolsVo.closeGoods === 0
              this.closeComment = data.roomToolsVo.closeComment === 0
              this.closeReplay = data.roomToolsVo.closeReplay === 0
              this.closeShare = data.roomToolsVo.closeShare === 0
              this.closeKf = data.roomToolsVo.closeKf === 0
            }
          })
        }
      })
    },
    // 表单提交
    dataFormSubmit () {
      this.dataForm.roomToolsVo.closeLike = this.closeLike ? 0 : 1
      this.dataForm.roomToolsVo.closeGoods = this.closeGoods ? 0 : 1
      this.dataForm.roomToolsVo.closeComment = this.closeComment ? 0 : 1
      this.dataForm.roomToolsVo.closeReplay = this.closeReplay ? 0 : 1
      this.dataForm.roomToolsVo.closeShare = this.closeShare ? 0 : 1
      this.dataForm.roomToolsVo.closeKf = this.closeKf ? 0 : 1
      // this.dataForm.startTime = this.dataForm.dateRange[0]
      // this.dataForm.endTime = this.dataForm.dateRange[1]
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.isSubmit) {
            return
          }
          let startTime = this.dataForm.startTime
          let endTime = this.dataForm.endTime
          this.dataForm.startTime = this.dataForm.startTime && this.startTimeValue ? this.dataForm.startTime + ' ' + this.startTimeValue + ':00' : ''
          this.dataForm.endTime = this.dataForm.endTime && this.endTimeValue ? this.dataForm.endTime + ' ' + this.endTimeValue + ':00' : ''
          this.isSubmit = true
          this.$http({
            url: this.$http.adornUrl('/live/liveRoom'),
            method: this.dataForm.id ? 'put' : 'post',
            data: this.$http.adornData(this.dataForm)
          }).then(({ data }) => {
            this.isSubmit = true
            this.$message({
              message: this.$i18n.t('publics.operation'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.visible = false
                this.back()
              }
            })
          }).catch(() => {
            console.log('boom')
            this.isSubmit = false
          })
          this.dataForm.startTime = startTime
          this.dataForm.endTime = endTime
        }
      })
    },
    back () {
      this.$router.push({
        path: '/live-liveRoom'
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.mod-live-room-add-or-update {
  .live-tips {
    color: #aaa;
    line-height: 20px;
    margin: 0;
    padding: 0;
  }
  .live-time-row {
    display: flex;
    .tip {
      margin: 0 10px;
    }
  }
  .el-input.el-input--small {
    width: 524px;
  }
  .el-date-editor.el-input.el-input--small {
    width: 245px;
  }
}

.liveTime ::v-deep .el-form-item__label:before{
  content: '*';
  color: #F56C6C;
  margin-right: 4px;
}
.el-date-editor >>>.el-input__suffix>.el-icon-circle-close {
  display: none
}
.el-date-editor >>>.el-input__suffix>.el-icon-circle-check {
  display: none
}

</style>
