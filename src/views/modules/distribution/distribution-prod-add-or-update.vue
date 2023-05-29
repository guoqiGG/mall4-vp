<template>
  <div class="mod-distribution-prod-add-or-update">
    <el-dialog
      :title="distributionProdId==2 ? $t('seckill.edit') :distributionProdId==1? $t('seckill.view'):$t('seckill.newAdd')"
      :close-on-click-modal="false"
      :visible.sync="visible"
    >
      <el-form @submit.native.prevent
        :model="dataForm"
        :rules="dataRule"
        ref="dataForm"
        @keyup.enter.native="dataFormSubmit ()"
        :label-width="this.$i18n.t('language') === 'language' ? '130px' : '90px'"
        :disabled="distributionProdId!==1?false:true"
      >
        <el-form-item :label="$t('groups.relatedProducts')">
          <div v-if="prodData[0]!=null">
            <el-card :body-style="{ padding: '0px' }" style="height: 160px;width: 120px">
              <prod-pic
                height="104px"
                width="100%"
                :pic="prodData[0].pic"
              ></prod-pic>
              <div class="card-prod-bottom" v-if="distributionProdId!==1">
                <span class="card-prod-name">{{prodData[0].prodName}}</span>
                <div
                class="default-btn text-btn"
                @click="deleteRelation"
                style="margin-right:5px;"
                >{{ $t("text.delBtn") }}</div
              >
              </div>
            </el-card>
          </div>
          <div v-if="prodData[0]==null" class="default-btn" @click="addProd">{{$t('product.select')}}</div>
        </el-form-item>
<!--
        <el-form-item :label="$t('marketing.systemSetup')" prop="defaultReward">
          <el-radio-group v-model="dataForm.defaultReward">
            <el-radio :label="0">{{$t('marketing.noUse')}}</el-radio>
            <el-radio :label="1">{{$t('marketing.use')}}</el-radio>
          </el-radio-group>
        </el-form-item> -->

        <div v-if="dataForm.defaultReward === 0">
          <el-form-item :label="$t('marketing.rewardRatio')" prop="awardProportion">
            <el-radio-group v-model="dataForm.awardProportion">
              <el-radio :label="0" >{{$t('marketing.proporteward')}}</el-radio>
              <el-radio :label="1">{{$t('marketing.rewardByFixedValue')}}</el-radio>
            </el-radio-group>
          </el-form-item>

          <el-form-item label="合伙人奖励" prop="parentAwardSet">
            <el-radio-group v-model="dataForm.parentAwardSet">
              <el-radio :label="0" disabled>{{$t('seckill.close')}}</el-radio>
              <el-radio :label="1">{{$t('seckill.open')}}</el-radio>
            </el-radio-group>
          </el-form-item>

          <el-form-item :label="dataForm.awardNumberSet === 1 ? $t('marketing.amountSetting') : '团长奖励'" prop="awardNumbers" :rules="[{ validator: validateAwardNumber, trigger: 'change' }]">
            <div v-if="dataForm.awardNumberSet === 0">
              <el-input v-model="dataForm.awardNumbers" size="small" :precision="2" :min="0" style="width:200px">
                <template slot="append">
                  <span v-if="dataForm.awardProportion === 1">{{$t('distribution.dbcTip2')}}</span>
                  <span v-else>%</span>
                </template>
              </el-input>

              <!-- <span
                v-if="dataForm.parentAwardSet === 1"
              >&nbsp; {{$t('marketing.inviterRewardAmount')}}:</span>
              <el-input
                v-model="dataForm.parentAwardNumbers"
                :precision="2"
                :min="0"
                size="small"
                v-if="dataForm.parentAwardSet === 1"
                style="width:200px"
              >
                <template slot="append">
                  <span v-if="dataForm.awardProportion === 1">{{$t('distribution.dbcTip2')}}</span>
                  <span v-else>%</span>
                </template>
              </el-input> -->
            </div>
          </el-form-item>
          <el-form-item
            v-if="dataForm.parentAwardSet === 1"
            label="合伙人奖励"
            prop="parentAwardNumbers"
            :rules="[{ validator: validateAwardNumber, trigger: 'change' }]"
            >
            <el-input v-model="dataForm.parentAwardNumbers" size="small" :precision="2" :min="0" style="width:200px">
              <template slot="append">
                <span v-if="dataForm.awardProportion === 1">{{ $t('distribution.dbcTip2') }}</span>
                <span v-else>%</span>
              </template>
            </el-input>
          </el-form-item>
        </div>
        <el-form-item :label="$t('product.status')" prop="state">
          <el-radio-group v-model="dataForm.state">
            <el-radio :label="1">{{$t('prodList.onShelf')}}</el-radio>
            <el-radio :label="0">{{$t('prodList.offShelf')}}</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <div class="default-btn" @click="visible = false">{{$t('seckill.close')}}</div>
        <div v-if="distributionProdId!==1" class="default-btn primary-btn" @click="dataFormSubmit()">{{$t('remindPop.confirm')}}</div>
      </span>
    </el-dialog>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addProdVisible" ref="addProd" @refreshDiscountProds="selectDiscountProds"></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from './discountProd-add-or-update'
import ProdPic from '@/components/prod-pic'
export default {
  data () {
    return {
      dataForm: {
        'prodId': 0,
        'distributionAmount': 0,
        'awardId': 0,
        'state': 1,
        'defaultReward': 0,
        'awardProportion': 0,
        'awardNumberSet': 1,
        'awardNumbers': '',
        'parentAwardNumbers': '',
        'parentAwardSet': 0
      },
      distributionProdId: 1,
      resourcesUrl: process.env.VUE_APP_RESOURCES_URL,
      levelData: [],
      prodData: [],
      isSubmit: false,
      addProdVisible: false,
      visible: false,
      isInviterReward: true,
      dataRule: {

      },
      isNeedInitAwardNumber: false
    }
  },
  computed: {
    listenChange() {
      return {
        isNeedInitAwardNumber: this.isNeedInitAwardNumber,
        prodData: this.prodData,
        awardProportion: this.dataForm.awardProportion
      }
    }
  },
  components: {
    AddOrUpdate,
    ProdPic
  },
  watch: {
    'listenChange'(newVal, oldVal) {
      if (newVal.isNeedInitAwardNumber !== oldVal.isNeedInitAwardNumber) return
      this.dataForm.awardNumbers = 0
    },
  },
  methods: {
    init (data) {
      this.visible = true
      this.isSubmit = false
      if (data) {
        this.isNeedInitAwardNumber = true
        this.dataForm = data
        this.prodData[0] = this.dataForm.product
      } else {
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          this.dataForm.defaultReward = 0
          this.dataForm.awardProportion = 0
          this.dataForm.awardNumberSet = 0
          this.dataForm.parentAwardSet = 1
          this.dataForm.awardNumbers = 0
          this.dataForm.parentAwardNumbers = 0
          this.dataForm.state = 1
          this.levelData = []
          this.prodData = []
        })
        this.isNeedInitAwardNumber = false
      }
    },

    // 删除关联商品数据
    deleteRelation () {
      this.dataForm.prodId = null
      this.prodData = []
    },
    // 打开选择商品
    addProd () {
      this.addProdVisible = true
      this.$nextTick(() => {
        this.$refs.addProd.init(0, this.prodData)
      })
    },
    // 商品选择回调
    selectDiscountProds (prods) {
      if (prods) {
        this.$nextTick(() => {
          this.$http({
            url: this.$http.adornUrl('/platform/distributionProd/count'),
            method: 'get',
            params: this.$http.adornParams({
              prodId: prods[0].prodId
            })
          }).then(({ data }) => {
            if (data === 0) {
              this.dataForm.prodId = prods[0].prodId
              this.$set(this.prodData, 0, prods[0])
            } else {
              this.$message({
                message: '该商品已经在分销商品列表中',
                type: 'warning'
              })
            }
          })
        })
      }
    },

    // 表单提交
    dataFormSubmit () {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.dataForm.awardNumberSet === 1) {
            // 创建json存入奖励表
            let awardNumberjson = []
            let parentAwardNumberjson = []
            this.levelData.forEach((item, index) => {
              awardNumberjson.push(Number.parseFloat(item.awardNumber).toFixed(2))
              parentAwardNumberjson.push(Number.parseFloat(item.parentAwardNumber).toFixed(2))
            })
            this.dataForm.awardNumbers = JSON.stringify(awardNumberjson)
            this.dataForm.parentAwardNumbers = JSON.stringify(parentAwardNumberjson)
          }
          let param = this.dataForm
          if (this.isSubmit) {
            return false
          }
          this.isSubmit = true
          this.$http({
            url: this.$http.adornUrl(`/platform/distributionProd`),
            method: param.distributionProdId ? 'put' : 'post',
            data: this.$http.adornData(param)
          }).then(({ data }) => {
            this.$message({
              message: this.$i18n.t('remindPop.success'),
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.visible = false
                this.$emit('refreshDataList')
              }
            })
          }).catch((e) => {
            this.isSubmit = false
          })
        }
      })
    },
    awardNumberSetChange (val) {
      if (val === 0) {
        this.dataForm.awardNumbers = 0
        this.dataForm.parentAwardNumbers = 0
      }
    },
    validateAwardNumber (rule, value, callback) {
      if (this.dataForm.awardProportion === 1) {
        if (this.prodData[0].price * 0.5 < parseFloat(value)) {
          callback(new Error('奖励金额不能超过商品价格的一半'))
        } else {
          callback()
        }
      } else {
        if (parseFloat(value) > 50) {
          callback(new Error('奖励比例不能大于50%'))
        } else {
          callback()
        }
      }
    }
  }
}
</script>
<style lang="scss" scoped>
.mod-distribution-prod-add-or-update {
  .card-prod-bottom {
    display: flex;
    align-items: center;
  }
}

</style>
