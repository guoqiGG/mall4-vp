<template>
    <el-dialog :close-on-click-modal="false" :visible.sync="visible" class="store-editor-pop" title="订单退款">
        <div class="orderInfo">
            <div>订单编号：{{ orderNumber }}</div>
            <div>买家昵称：{{ nickName }}</div>
        </div>

        <div class="shopInfo" style="margin-top:20px;">
            <div class="goods-list">
                <el-table :data="prodList" border>
                    <el-table-column :label="this.$i18n.t('order.product')">
                        <template slot-scope="scope">
                            <div class="df" style="display: flex;align-items: center;">
                                <prod-pic :height="60" :width="60" class="prod-pic" :pic="scope.row.pic"></prod-pic>
                                <div class="name" style="margin-left:20px;">
                                    <div>
                                        <span v-if="scope.row.giveawayOrderItemId" class="gift-icon">{{
                                            $t("order.giveawayPord") }}</span>
                                        <span>{{ scope.row.prodName }}</span>
                                        <span class="sku">{{ scope.row.skuName }}</span>
                                    </div>
                                    <div class="order-status" v-if="scope.row.returnMoneySts &&
                                        scope.row.returnMoneySts < 6 &&
                                        scope.row.returnMoneySts > 0
                                        ">
                                        {{
                                            [
                                                "",
                                                $t("refundOrderDetail.buyerApply"),
                                                $t("order.sellerAccepts"),
                                                $t("refundOrderDetail.buyerDelivery"),
                                                $t("order.sellerReceivesGoods"),
                                                $t("order.refundsuccessfully"),
                                            ][scope.row.returnMoneySts]
                                        }}
                                    </div>
                                </div>
                            </div>

                        </template>
                    </el-table-column>
                    <el-table-column prop="price" :label="this.$i18n.t('order.unitPrice')" width="180" align="center">
                        <template slot-scope="scope">
                            <span>{{ scope.row.giveawayOrderItemId ? '-' : scope.row.price }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="count" :label="this.$i18n.t('order.quantity')" width="180" align="center">
                        <template slot-scope="scope">
                            <span>{{ scope.row.prodCount }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="count" :label="this.$i18n.t('order.preferentialAmount')" width="180"
                        align="center">
                        <template slot-scope="scope">
                            <span>{{ scope.row.giveawayOrderItemId ? '-' : scope.row.shareReduce }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="totalPrice" :label="this.$i18n.t('order.totalPrice')" width="180" align="center">
                        <template slot-scope="scope">
                            <span>{{ scope.row.giveawayOrderItemId ? '-' : scope.row.productTotalAmount }}</span>
                        </template>
                    </el-table-column>
                </el-table>
            </div>
        </div>

        <el-form style="margin-top:20px;" @submit.native.prevent :model="dataForm" ref="dataForm"
            @keyup.enter.native="dataFormSubmit()" label-width="120px" size="small">
            <!-- <el-form-item label="申请类型" prop="applyType">
                <el-radio-group v-model="dataForm.applyType" disabled>
                    <el-radio :label="1">仅退款</el-radio>
                    <el-radio :label="2">退货退款</el-radio>
                </el-radio-group>
            </el-form-item> -->

            <el-form-item label="退款单类型" prop="refundType">
                <el-radio-group v-model="dataForm.refundType" disabled>
                    <el-radio :label="1">整单退款</el-radio>
                    <el-radio :label="2">单个物品退款</el-radio>
                </el-radio-group>
            </el-form-item>

            <el-form-item label="货物状态" prop="isReceiver">
                <el-radio-group v-model="dataForm.isReceiver">
                    <el-radio :label="1">已收到货</el-radio>
                    <el-radio :label="0">未收到货</el-radio>
                </el-radio-group>
            </el-form-item>

            <el-form-item label="申请原因" prop="buyerReason">
                <el-radio-group v-model="dataForm.buyerReason">
                    <el-radio label="拍错">拍错</el-radio>
                    <el-radio label="多拍">多拍</el-radio>
                    <el-radio label="不喜欢">不喜欢</el-radio>
                </el-radio-group>
            </el-form-item>

            <el-form-item label="退款金额" prop="refundAmount">
                <el-input-number v-model="dataForm.refundAmount" disabled placeholder="退款金额" controls-position="right"
                    :min="0.01"></el-input-number>
            </el-form-item>
        </el-form>

        <span slot="footer" class="dialog-footer">
            <div class="default-btn" @click="visible = false">{{
                $t("remindPop.cancel")
            }}</div>
            <div class="default-btn primary-btn" @click="dataFormSubmit()">{{
                $t("remindPop.confirm") }}</div>
        </span>
    </el-dialog>
</template>
  
<script>
import ProdPic from '@/components/prod-pic'
export default {
    components: {
        ProdPic
    },
    data() {
        return {
            isSubmit: false,
            visible: false,
            orderNumber: '',// 订单编号
            nickName: '',
            dataForm: {
                applyType: 1, //申请类型(1:仅退款 2退款退货)  
                refundType: 2,//1:整单退款,2:单个物品退款
                orderItemId: '',//订单项ID（单个物品退款时使用）	
                goodsNum: null, //退款数量（0或不传值则为全部数量）
                isReceiver: 1,//货物状态(1:已收到货 0:未收到货)
                buyerReason: '不喜欢',//申请原因(下拉选择)
                refundAmount: null, // 退款金额
                buyerMobile: null, // 买家手机号
                userId: null,
            },
            prodList: [], // 数组
            prodListObject: {}// 对象
        }
    },

    methods: {
        init(orderNumber, nickName, tel, refundType, prodList) {
            this.visible = true
            this.$nextTick(() => {
                this.$refs.dataForm.resetFields()
                this.orderNumber = orderNumber
                this.nickName = nickName
                this.dataForm.refundType = refundType

                if (refundType == 1) { //1:整单退款
                    this.prodList = prodList
                    let totalPrice = 0
                    prodList.forEach(e => {
                        totalPrice += e.actualTotal
                    })
                    this.dataForm.refundAmount = totalPrice
                    this.dataForm.goodsNum = prodList.prodCount
                    this.dataForm.userId = prodList.userId
                    this.dataForm.buyerMobile = tel
                } else if (refundType == 2) { //2:单个物品退款
                    let list = []
                    list.push(prodList)
                    this.prodList = list
                    this.dataForm.refundAmount = prodList.actualTotal
                    this.dataForm.orderItemId = prodList.orderItemId
                    this.dataForm.goodsNum = prodList.prodCount
                    this.dataForm.userId = prodList.userId
                    this.dataForm.buyerMobile = tel
                }
            })
        },

        // 表单提交
        dataFormSubmit() {
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    if (this.isSubmit) {
                        return false
                    }
                    this.isSubmit = true
                    this.$http({
                        url: this.$http.adornUrl('/platform/shopCompany/apply'),
                        method: 'post',
                        data: this.$http.adornData({
                            orderNumber: this.orderNumber, //订单退款
                            refundType: this.dataForm.refundType,//1:整单退款,2:单个物品退款
                            orderItemId: this.dataForm.orderItemId,//订单项ID（单个物品退款时使用）	
                            goodsNum: this.dataForm.goodsNum, //退款数量（0或不传值则为全部数量）
                            applyType: this.dataForm.applyType, //申请类型(1:仅退款 2退款退货)
                            isReceiver: this.dataForm.isReceiver,//货物状态(1:已收到货 0:未收到货)
                            buyerReason: this.dataForm.buyerReason,//申请原因(下拉选择)
                            refundAmount: this.dataForm.refundAmount, // 退款金额
                            buyerMobile: this.dataForm.buyerMobile,
                            userId: this.dataForm.userId
                        })
                    }).then(({ data }) => {
                        this.$message({
                            message: '订单发起退款成功,请在退款管理中进行退款操作',
                            type: 'success',
                            duration: 1500,
                        })
                        this.visible=false
                    }).catch((e) => {
                        this.isSubmit = false
                    })
                }
            })
        },
        /**
    * 确定退款
    */
        returnMoneyHandle(refundId, refundSn) {
            this.$http({
                url: this.$http.adornUrl2(`/order/refund/process`),
                method: 'put',
                data: this.$http.adornData({
                    refundId: refundId,
                    refundSts: 2,
                    refundSn: refundSn,
                    rejectMessage: '',
                    sellerMsg: '同意'
                })
            }).then(({ data }) => {
                console.log(data)
                this.$message({
                    message: '同意退款',
                    type: 'success',
                    duration: 1500
                })
                this.refundRequest(refundSn)
            }).catch(() => {
            })
        },

        /**
    * 退款请求（发放退款）
    */
        refundRequest(refundSn) {
            this.$http({
                url: this.$http.adornUrl2(`/order/refund/refundRequest`),
                method: 'put',
                data: { 'refundSn': refundSn }
            }).then(() => {
                this.$message({
                    message: '发放退款',
                    type: 'success',
                    duration: 1500,
                    onClose: () => {
                    }
                })
                this.visible = false
            })
        },
    },
}
</script>
  
<style lang="scss">
.el-form-item {
    margin-bottom: 30px;
}

.pcd .el-form-item__label:before {
    content: '*';
    color: #F56C6C;
    margin-right: 4px;
}

.orderInfo {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: flex-start;
}

.orderInfo div {
    margin-right: 30px;
}
</style>
  