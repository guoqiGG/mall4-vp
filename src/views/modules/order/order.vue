<template>
  <div class="mod-order-order">
    <div class="search-bar">
      <el-form @submit.native.prevent :inline="true" :model="dataForm" @keyup.enter.native="getDataList(this.page)"
        size="small">
        <div class="input-row">
          <!-- &nbsp;&nbsp;&nbsp; -->
          <el-form-item :label="this.$i18n.t('order.number') + '：'">
            <el-input v-model="dataForm.orderNumber" :placeholder="this.$i18n.t('order.number')" clearable
              size="small"></el-input>
          </el-form-item>
          <!-- <el-form-item label="商品名称：">
            <el-input v-model="dataForm.prodName" placeholder="请输入商品名称" clearable></el-input>
          </el-form-item>-->
          <el-form-item :label="this.$i18n.t('prodList.shopName') + '：'">
            <el-input style="min-width: 210px;" v-model="dataForm.shopName"
              :placeholder="this.$i18n.t('prodList.inputShopName')" clearable size="small"></el-input>
          </el-form-item>
          <el-form-item :label="this.$i18n.t('order.status') + '：'">
            <template>
              <el-select v-model="status" clearable :placeholder="this.$i18n.t('order.statusMsg')" size="small"
                @change="orderStatus">
                <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value"></el-option>
              </el-select>
            </template>
          </el-form-item>
          <!-- &nbsp;&nbsp;&nbsp; -->
          <el-form-item :label="this.$i18n.t('order.afterSalesStatus') + '：'">
            <template>
              <el-select style="min-width: 230px;" v-model="dataForm.refundStatus" clearable
                :placeholder="this.$i18n.t('order.pleSelAfterSalesSta')" size="small">
                <el-option v-for="item in refund" :key="item.value" :label="item.label" :value="item.value"></el-option>
              </el-select>
            </template>
          </el-form-item>
          <el-form-item :label="this.$i18n.t('order.paymentMethod') + '：'">
            <template>
              <el-select v-model="dataForm.payType" clearable :placeholder="this.$i18n.t('order.paymentMethod')"
                size="small">
                <el-option v-for="item in payType" :key="item.value" :label="item.label" :value="item.value"></el-option>
              </el-select>
            </template>
          </el-form-item>
          <el-form-item :label="this.$i18n.t('order.orderType') + '：'">
            <template>
              <el-select v-model="dataForm.orderType" clearable :placeholder="this.$i18n.t('order.orderType')"
                size="small">
                <el-option v-for="item in orderType" :key="item.value" :label="item.label"
                  :value="item.value"></el-option>
              </el-select>
            </template>
          </el-form-item>
          <el-form-item :label="this.$i18n.t('order.orderMold') + ':'" :label-width="lang === 'en' ? '145px' : '85px'">
            <el-select v-model="dataForm.orderMold" clearable :placeholder="this.$i18n.t('order.pleaseSelectOrderMold')"
              size="small">
              <el-option v-for="item in orderMold" :key="item.value" :label="item.label" :value="item.value"></el-option>
            </el-select>
          </el-form-item>
          <!-- &nbsp;&nbsp;&nbsp; -->
          <el-form-item :label="this.$i18n.t('order.theRecipientSName') + '：'">
            <el-input style="min-width: 240px;" v-model="dataForm.receiver"
              :placeholder="this.$i18n.t('order.pleaseEnRecipName')" clearable size="small"></el-input>
          </el-form-item>
          <el-form-item :label="this.$i18n.t('order.contactTel') + '：'">
            <el-input style="min-width: 340px;" v-model="dataForm.mobile"
              :placeholder="this.$i18n.t('order.pleaseEnterNumber')" clearable type="number" size="small"
              class="myinput-appearance"></el-input>
          </el-form-item>
          <el-form-item :label="this.$i18n.t('order.logisticsType') + '：'">
            <template>
              <el-select style="min-width: 210px;" v-model="dataForm.dvyType" clearable
                :placeholder="this.$i18n.t('order.selectLogType')" size="small">
                <el-option v-for="item in dvyType" :key="item.value" :label="item.label" :value="item.value"></el-option>
              </el-select>
            </template>
          </el-form-item>
          <el-form-item :label="this.$i18n.t('order.pickUpPoint') + '：'">
            <el-input v-model="dataForm.stationName" :placeholder="this.$i18n.t('order.pickUpPoint')" clearable
              size="small"></el-input>
          </el-form-item>
          <!-- &nbsp;&nbsp;&nbsp; -->
          <el-form-item :label="this.$i18n.t('order.createTime') + '：'">
            <el-date-picker size="small" v-model="dateRange" type="datetimerange" range-separator="—"
              value-format="yyyy-MM-dd HH:mm:ss" :start-placeholder="this.$i18n.t('date.start')"
              :end-placeholder="this.$i18n.t('date.end')"></el-date-picker>
            <div style="margin-left: 20px;" class="default-btn" @click="setDateRange(1)">{{
              $t("date.t")
            }}</div>
            <div class="default-btn" @click="setDateRange(2)">{{
              $t("date.y")
            }}</div>
            <div class="default-btn" @click="setDateRange(3)">{{
              $t("date.n")
            }}</div>
            <div class="default-btn" @click="setDateRange(4)">{{
              $t("th")
            }}</div>
          </el-form-item>
          <el-form-item>
            <div class="default-btn primary-btn" @click="searchChange(true)">{{ $t("order.query") }}</div>
            <div class="default-btn" @click="getSoldExcel()">{{ $t("order.export") }}</div>
            <div class="default-btn" @click="clear()">{{ $t("product.reset") }}</div>
          </el-form-item>
        </div>
        <!-- <div class="operation-box">
          <el-form-item>
            <el-button
              class="operation-btn"
              type="primary"
              icon="el-icon-search"
              size="small"
              @click="searchChange()"
            >查询</el-button>
          </el-form-item>
          <el-form-item>
            <el-button
              class="operation-btn"
              @click="getSoldExcel()"
              size="small"
              icon="el-icon-document-checked"
            >导出</el-button>
          </el-form-item>
          <el-form-item>
            <el-button class="operation-btn" icon="el-icon-delete" size="small" @click="clear()">清空</el-button>
          </el-form-item>
        </div>-->
      </el-form>
      <el-form @submit.native.prevent :inline="true" :model="dataForm1" size="small">
        <div class="input-row">
          <!-- &nbsp;&nbsp;&nbsp; -->
          <el-form-item label="商品名称：">
            <template>
              <el-select style="min-width: 230px;" v-model="dataForm1.prodId" clearable placeholder="商品名称" size="small">
                <el-option v-for="item in prodDataList" :key="item.prodId" :label="item.prodName"
                  :value="item.prodId"></el-option>
              </el-select>
            </template>
          </el-form-item>
          <el-form-item>
            <div class="default-btn" @click="getSoldExcelByProd()">按商品导出</div>
          </el-form-item>
        </div>
      </el-form>
    </div>
    <div class="main-container main">
      <div class="content">
        <!-- 导航 -->
        <div :class="['order-status-nav', 'clearfix', showHeadScroll ? 'status-nav-bottom' : '']">
          <ul class="nav-list clearfix">
            <li :class="['nav-item', sts == 0 ? 'selected' : '']" data-sts="0" @click="selectNav($event)">
              {{ $t("date.a") }}
            </li>
            <li :class="['nav-item', sts == 1 ? 'selected' : '']" data-sts="1" @click="selectNav($event)">
              {{ $t("order.pendingPayment") }}
            </li>
            <li :class="['nav-item', sts == 2 ? 'selected' : '']" data-sts="2" @click="selectNav($event)">
              {{ $t("order.toBeShipped") }}
            </li>
            <li :class="['nav-item', sts == 3 ? 'selected' : '']" data-sts="3" @click="selectNav($event)">
              {{ $t("order.pendingReceipt") }}
            </li>
            <li :class="['nav-item', sts == 5 ? 'selected' : '']" data-sts="5" @click="selectNav($event)">
              {{ $t("order.successfulTransaction") }}
            </li>
            <li :class="['nav-item', sts == 6 ? 'selected' : '']" data-sts="6" @click="selectNav($event)">
              {{ $t("order.transactionFailed") }}
            </li>
            <li :class="['nav-item', sts == 7 ? 'selected' : '']" data-sts="7" @click="selectNav($event)">
              {{ $t("group.waitGroup") }}
            </li>
            <li v-if="status == 3" class="nav-item" @click="oneClickDelivery()">
              一键收货
            </li>
          </ul>
          <ul class="nav-right"></ul>
        </div>

        <!-- 列标题 -->
        <div :class="['tit', showHeadScroll ? sidebarFold ? 'fixed-top-fold' : 'fixed-top' : '']">
          <el-row style="width: 100%">
            <el-col :span="1" style="text-align: center;">
              <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll"
                @change="handleCheckAllChange"></el-checkbox>
            </el-col>
            <el-col :span="5">
              <span class="item product">{{ $t("group.prodInfo") }}</span>
            </el-col>
            <el-col :span="2" class="transaction-price">
              <span class="item">{{ $t("prodList.goodsPrice") }}/{{
                $t("order.quantity")
              }}</span>
            </el-col>
            <el-col :span="3" class="column-title">
              <span class="item">{{ $t("order.paymentAmount") }}</span>
            </el-col>
            <el-col :span="2" class="column-title">
              <span class="item">{{ $t("order.paymentMethod") }}</span>
            </el-col>
            <el-col :span="3" class="column-title">
              <span class="item">团长</span>
            </el-col>
            <el-col :span="2" class="column-title">
              <span class="item">{{ $t("order.buyerConsignee") }}</span>
            </el-col>
            <el-col :span="2" class="column-title">
              <span class="item">{{ $t("order.orderStatus") }}</span>
            </el-col>
            <el-col :span="2" class="column-title">
              <span class="item">{{ $t("order.afterSalesStatus") }}</span>
            </el-col>
            <el-col :span="2" class="column-title">
              <span class="item">{{ $t("order.operation") }}</span>
            </el-col>
          </el-row>
        </div>

        <el-checkbox-group v-model="orderNumberList" @change="handleCheckedChange">
          <div class="prod" v-for="order in dataList" :key="order.orderId">
            <div class="prod-tit">
              <span>{{ $t("order.number") }}：{{ order.orderNumber }}</span>
              <span>{{ $t("order.createTime") }}：{{ order.createTime }}</span>
              <span>{{ $t("prodList.shopName") }}：{{ order.shopName }}</span>
              <!-- <span>买家：19999999999</span>
            <span >联系电话：19999999999</span>-->
            </div>
            <div class="prod-cont">
              <el-row style="width: 100%">
                <el-col :span="1" style="height:100%;">
                  <div style="display: flex;align-items:center;justify-content: center;height:100%;">
                    <el-checkbox :label="order.orderNumber" :key="order.orderNumber"><br></el-checkbox>
                  </div>
                </el-col>
                <el-col :span="7" style="height: 100%">
                  <div class="item prod-item">
                    <div class="items name" v-for="orderItem in order.orderItems" :key="orderItem.orderItemId">
                      <!-- 商品信息 -->
                      <div class="order-prod-item-info">
                        <div class="info">
                          <div class="prod-image">
                            <prod-pic :pic="orderItem.pic"></prod-pic>
                          </div>
                          <div class="prod-name">
                            <div class="prod-con">
                              <div class="prod-name-txt">
                                {{ orderItem.prodName }}
                              </div>
                              <div v-if="orderItem.skuName" class="prod-name-sku">
                                {{ orderItem.skuName }}
                              </div>
                              <div class="order-status" v-if="order.orderType === 1 || order.orderType === 2">
                                {{
                                  order.orderType === 1
                                  ? $t("order.groupPurchaseOrder")
                                  : order.orderType === 2
                                    ? $t("order.spikeOrder")
                                    : ""
                                }}
                              </div>
                              <div class="order-status" v-if="order.orderMold === 1">
                                {{ $t("order.virtualOrder") }}
                              </div>
                              <div class="order-status" v-if="!orderItem.returnMoneySts ||
                                orderItem.returnMoneySts < 0 ||
                                orderItem.returnMoneySts > 5
                                ">
                                {{
                                  orderItem.status === 0 && order.status === 2
                                  ? $t("order.pendingReceipt")
                                  : [
                                    "",
                                    $t("order.pendingPayment"),
                                    $t("order.toBeShipped"),
                                    $t("order.pendingReceipt"),
                                    "",
                                    $t("order.successfulTransaction"),
                                    $t("order.transactionFailed"),
                                    $t("group.waitGroup"),
                                  ][order.status]
                                }}
                              </div>
                              <div class="order-status" v-else>
                                {{
                                  [
                                    "",
                                    $t("refundOrderDetail.buyerApply"),
                                    $t("order.sellerAccepts"),
                                    $t("refundOrderDetail.buyerDelivery"),
                                    $t("order.sellerReceivesGoods"),
                                    $t("order.refundsuccessfully"),
                                  ][orderItem.returnMoneySts]
                                }}
                              </div>
                              <div class="order-status" v-if="order.dvyType === 2 || order.dvyType === 4">
                                {{
                                  order.dvyType === 2
                                  ? $t("order.selfMention")
                                  : order.dvyType === 4
                                    ? $t("order.sameCityDelivery")
                                    : ""
                                }}
                              </div>
                              <!-- <span class="prod-info">{{orderItem.skuName}}</span> -->
                              <div class="order-status" v-if="orderItem.preSaleTime !== null">
                                {{ $t('order.EstimatedDeliveryTime') }}{{ orderItem.preSaleTime }}
                              </div>
                            </div>
                          </div>
                        </div>
                        <!-- 赠品信息 -->
                        <div v-if="orderItem.giveawayList" class="order-prod-item-give-con">
                          <div class="giveaway-item" v-for="(giveawayItem, giveIndex) in orderItem.giveawayList"
                            :key="giveIndex">
                            <div class="giveaway-name"> 【{{ $i18n.t('order.giveawayPord') }}】 {{ giveawayItem.prodName }}
                            </div>
                            <div class="giveaway-name giveaway-sku-name">{{ giveawayItem.skuName || '' }}</div>
                            <div class="giveaway-item-sku-count"> x{{ giveawayItem.prodCount }}</div>
                          </div>
                        </div>
                      </div>
                      <div class="prod-price">
                        <span>{{ orderItem.price.toFixed(2) }}</span>
                        <span>{{ orderItem.prodCount
                        }}{{ $t("marketing.item") }}</span>
                      </div>
                    </div>
                  </div>
                </el-col>
                <el-col :span="3" style="height: 100%;text-align: center;">
                  <div class="item">
                    <div>
                      <span class="totalprice">
                        {{ order.actualTotal.toFixed(2) }}
                        {{
                          order.score && order.score > 0
                          ? "+ " + order.score + $t("order.score")
                          : ""
                        }}
                      </span>
                      <span class="totalprice" v-if="order.freightAmount - order.platformFreeFreightAmount > 0">（{{
                        $t("order.includingFreight") }}：
                        {{ (order.freightAmount - order.platformFreeFreightAmount).toFixed(2) }}）</span>
                      <span>{{ $t("order.total") }} {{ order.productNums }}
                        {{ $t("order.piece") }}</span>
                    </div>
                  </div>
                </el-col>
                <el-col :span="2" style="height: 100%">
                  <div class="item">
                    <div>
                      <span v-if="(!order.payType && order.payType != 0) || order.status === 1">{{
                        $t("order.unpaid")
                      }}</span>
                      <span v-else>
                        {{
                          [
                            $t("order.pointsPayment"),
                            $t("order.wecProPay"),
                            $t("order.alipayPCPayment"),
                            $t("order.wechatScanCodePayment"),
                            $t("order.wechatH5Payment"),
                            $t("order.weclAccountPay"),
                            $t("order.alipayH5Payment"),
                            $t("order.alipayAPPPayment"),
                            $t("order.wechatAPPPayment"),
                            $t("order.balancePayment"),
                            $t("order.payPalPayment"),
                          ][order.payType]
                        }}
                      </span>
                    </div>
                  </div>
                </el-col>
                <!-- 团长信息 -->
                <el-col :span="3" style="height: 100%">
                  <div class="item">
                    <div v-if="order.distributionUserResult">
                      <span>{{ order.distributionUserResult.distributionName + '电话：' +
                        order.distributionUserResult.stationTel }}</span>
                      <span style="margin-top: 5px;">{{ '门店：' + order.distributionUserResult.distributionStationName
                      }}</span>
                      <span style="margin-top: 5px;">
                        地址：{{ order.distributionUserResult.province + order.distributionUserResult.city +
                          order.distributionUserResult.area + order.distributionUserResult.addr }}
                      </span>
                    </div>
                    <div v-else>
                      无
                    </div>
                  </div>
                </el-col>
                <el-col :span="2" style="height: 100%">
                  <div class="item">
                    <div class="buyer-info">
                      <div class="buyer-name">
                        {{ order.receiverName }}
                      </div>
                      <div class="buyer-phone">
                        {{ order.receiverMobile }}
                      </div>
                    </div>
                  </div>
                </el-col>
                <el-col :span="2" style="height: 100%">
                  <div class="item">
                    <!-- <span v-if="order.refundStatus === 1" size="small" type="danger">退款申请中</span> -->
                    <span>
                      <span v-if="order.status === 1" size="small" type="danger">{{ $t("order.pendingPayment") }}</span>
                      <span v-else-if="order.status === 2" size="small" type="danger">{{ $t("order.toBeShipped") }}</span>
                      <span v-else-if="order.status === 3" size="small" type="danger">{{ $t("order.pendingReceipt")
                      }}</span>
                      <span v-else-if="order.status === 7" size="small" type="danger">{{ $t("group.waitGroup") }}</span>
                      <span v-else-if="order.status === 5" size="small" type="danger">{{ $t("order.successfulTransaction")
                      }}</span>
                      <span v-else-if="order.status === 6" size="small">{{
                        $t("order.transactionFailed")
                      }}</span>
                    </span>
                  </div>
                </el-col>
                <el-col :span="2" style="height: 100%">
                  <div class="item">
                    <span v-if="order.refundStatus === 1" size="small" type="danger">{{ $t("order.requestARefund")
                    }}</span>
                    <span v-else-if="order.refundStatus === 2" size="small" type="danger">{{
                      $t("order.refundsuccessfully")
                    }}</span>
                    <span v-else-if="order.refundStatus === 3" size="small" type="danger">{{ $t("order.partialRefundSucc")
                    }}</span>
                    <span v-else-if="order.refundStatus === 4" size="small" type="danger">{{ $t("order.refundFailed")
                    }}</span>
                    <span v-else size="small" type="danger">{{
                      $t("order.noAfterSales")
                    }}</span>
                  </div>
                </el-col>
                <el-col :span="2" style="height: 100%">
                  <div class="item">
                    <div class="operate">
                      <!-- <button onclick="">打印订单</button><br> -->
                      <div class="default-btn text-btn" @click="addOrUpdateHandle(order.orderNumber)">{{
                        $t("order.seeDetails") }}</div>
                      <div class="default-btn text-btn" @click="toImbox(order)">{{ $t("order.contactBuyer") }}</div>
                      <div class="default-btn text-btn" @click="refundRoute(order.orderNumber)" v-if="order.refundStatus">
                        {{
                          $t("order.refundInformation") }}</div>
                    </div>
                  </div>
                </el-col>
              </el-row>
            </div>
            <!-- <div class="remark">
            <div class="buyer-remark">
              <span>备注:{{order.remarks}}</span>
            </div>
          </div>-->
          </div>
        </el-checkbox-group>
        <div class="empty" v-if="!dataList.length">
          {{ $t("order.noData") }}
        </div>
      </div>
    </div>
    <el-pagination v-if="dataList.length" @size-change="sizeChangeHandle" @current-change="currentChangeHandle"
      :current-page="page.currentPage" :page-sizes="[10, 20, 50, 100]" :page-size="page.pageSize" :total="page.total"
      layout="total, sizes, prev, pager, next, jumper"></el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <consignment-info v-if="consignmentInfoVisible" ref="consignmentInfo"
      @inputCallback="getWaitingConsignmentExcel"></consignment-info>
  </div>
</template>

<script>
import AddOrUpdate from './orderInfo'
import ConsignmentInfo from './consignment-info'
import moment from 'moment'
import ProdPic from '@/components/prod-pic'
import { isAuth } from '@/utils'
export default {
  name: 'order-order',
  data() {
    return {
      theData: null, // 保存上次点击查询的请求条件

      showHeadScroll: false,
      resizeProportion: 1,
      sts: 0,
      dataForm: {},
      dataForm1: {
        prodId: null
      },
      dateRange: [],
      status: null,
      options: [{
        value: 1,
        label: this.$i18n.t('order.pendingPayment')
      },
      {
        value: 2,
        label: this.$i18n.t('order.toBeShipped')
      },
      {
        value: 3,
        label: this.$i18n.t('order.pendingReceipt')
      },
      {
        value: 5,
        label: this.$i18n.t('order.successfulTransaction')
      },
      {
        value: 6,
        label: this.$i18n.t('order.transactionFailed')
      },
      {
        value: 7,
        label: this.$i18n.t('group.waitGroup')
      }],
      refund: [{
        value: 0,
        label: this.$i18n.t('order.noAfterSales')
      },
      {
        value: 1,
        label: this.$i18n.t('order.requestARefund')
      },
      {
        value: 2,
        label: this.$i18n.t('order.refundsuccessfully')
      },
      {
        value: 3,
        label: this.$i18n.t('order.partialRefundSucc')
      },
      {
        value: 4,
        label: this.$i18n.t('order.refundFailed')
      }],
      orderType: [{
        value: 0,
        label: this.$i18n.t('order.normalOrder')
      }, {
        value: 1,
        label: this.$i18n.t('order.groupPurchaseOrder')
      }, {
        value: 2,
        label: this.$i18n.t('order.spikeOrder')
      }],
      orderMold: [{
        value: 0,
        label: this.$i18n.t('order.physicalOrder')
      }, {
        value: 1,
        label: this.$i18n.t('order.virtualOrder')
      }],
      dvyType: [{
        value: 1,
        label: this.$i18n.t('order.expressDelivery')
      },
      {
        value: 2,
        label: this.$i18n.t('order.selfMention')
      },
      {
        value: 3,
        label: this.$i18n.t('order.noNeedRequired')
      },
      {
        value: 4,
        label: this.$i18n.t('order.sameCityDelivery')
      }],
      payType: [{
        value: 0,
        label: this.$i18n.t('order.pointsPayment')
      }, {
        value: 1,
        label: this.$i18n.t('order.weixinPay')
      }, {
        value: 2,
        label: this.$i18n.t('order.aliPay')
      }, {
        value: 3,
        label: this.$i18n.t('publics.balancePay')
      }, {
        value: 4,
        label: this.$i18n.t('order.payPalPayment')
      }],
      resourcesUrl: process.env.VUE_APP_RESOURCES_URL,
      dataList: [],
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      },
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      consignmentInfoVisible: false,
      lang: localStorage.getItem('bbcLang') || 'zh_CN',

      oldPar: {},
      checkAll: false,
      isIndeterminate: false,
      orderNumberList: [],
      orderIdList: [],
      prodDataList: []
    }
  },
  computed: {
    // 二级菜单折叠状态
    sidebarFold: {
      get() { return this.$store.state.common.sidebarFold },
      set(val) { this.$store.commit('common/updateSidebarFold', val) }
    }
  },
  components: {
    AddOrUpdate,
    ConsignmentInfo,
    ProdPic
  },
  created() {
    // 首页跳转状态参数
    this.activeName = this.$route.query.status ? String(this.$route.query.status) : '0'
    this.sts = this.$route.query.status || 0
    this.status = this.sts === 0 ? null : this.sts

    // 携带参数查询
    this.getDataList(this.page, this.$route.query)
    this.getProdList()
  },
  activated() {
    // 携带参数查询
    var query = this.$route.query
    if (Object.keys(query).length > 0) {
      this.getDataList(this.page, query)
    }
  },
  mounted() {
    // 监听页面滚动
    window.addEventListener('scroll', this.scrollToTop)

    this.resizeProportion = window.outerHeight / window.innerHeight
    window.addEventListener('resize', function () {
      this.resizeProportion = window.outerHeight / window.innerHeight
    })
  },
  methods: {
    handleCheckAllChange(val) {
      console.log(val)
      this.orderNumberList = val ? this.orderIdList : []
      this.indeterminate = false
    },
    handleCheckedChange(val) {
      console.log(val)
      let checkedCount = val.length
      this.checkAll = checkedCount === this.orderIdList.length
      this.isIndeterminate = checkedCount > 0 && checkedCount < this.orderIdList.length;
    },
    // 一键发货
    oneClickDelivery() {
      console.log(1111, this.orderNumberList.length)
      if (!this.orderNumberList.length) {
        this.$message.error({
          message: '请选择待收货的订单',
          type: 'success',
          duration: 1500,
          onClose: () => {
          }
        })
        return
      }
      this.$http({
        url: this.$http.adornUrl('/platform/order/receipt/list'),
        method: 'put',
        data: this.$http.adornData({
          orderNumberList: this.orderNumberList
        })
      }).then((data) => {
        this.$message.success({
          message: '一键收货成功',
          type: 'success',
          duration: 1500,
          onClose: () => {
          }
        })
        this.getDataList(this.page, {}, 1)
      })
    },
    /**
     * 页面滚动事件
     */
    scrollToTop() {
      let scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop
      this.showHeadScroll = scrollTop > (400 * this.resizeProportion)
    },

    /**
     * 获取数据列表
     */
    getDataList(page, params, type = 0, newData = false) {
      let par = Object.assign(
        {
          'orderNumber': this.dataForm.orderNumber,
          // 'prodName': this.dataForm.prodName,
          'shopName': this.dataForm.shopName,
          'orderType': this.dataForm.orderType,
          'orderMold': this.dataForm.orderMold,
          'payType': this.dataForm.payType,
          'receiver': this.dataForm.receiver,
          'mobile': this.dataForm.mobile,
          'status': this.status,
          'dvyType': this.dataForm.dvyType,
          'stationName': this.dataForm.stationName,
          'refundStatus': this.dataForm.refundStatus,
          'startTime': this.dateRange === null ? null : this.dateRange[0], // 开始时间
          'endTime': this.dateRange === null ? null : this.dateRange[1] // 结束时间
        },
        params
      )
      if (type === 0) {
        this.oldPar = par
      } else {
        par = this.oldPar
      }
      if (newData || !this.theData) {
        this.theData = par
      } else {
        this.theData.status = this.status
      }
      page = (page === undefined ? this.page : page)
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/platform/order/page'),
        method: 'get',
        params: this.$http.adornParams(
          Object.assign(
            {
              current: page == null ? this.page.currentPage : page.currentPage,
              size: page == null ? this.page.pageSize : page.pageSize
            },
            this.theData
          ), false
        )
      }).then(({ data }) => {
        this.dataList = data.records
        this.orderIdList = []
        if (this.status !== 3) {
          this.checkAll = false
          this.isIndeterminate = false
          this.orderNumberList = []
        }
        if (data.records) {
          data.records.forEach((e, i) => {
            this.orderIdList[i] = e.orderNumber
          });
        } else {
          this.orderIdList = []
        }
        // console.log(this.orderIdList)
        this.page.total = data.total
        this.sts = !this.status ? 0 : this.status
        this.dataListLoading = false
      })
    },
    // 每页数
    sizeChangeHandle(val) {
      this.page.pageSize = val
      this.page.currentPage = 1
      this.getDataList(this.page, {}, 1)
    },
    // 当前页
    currentChangeHandle(val) {
      this.page.currentPage = val
      this.getDataList(this.page, {}, 1)
    },

    /**
     * 导航选择状态
     */
    selectNav(e) {
      var sts = e.currentTarget.dataset.sts
      this.sts = parseInt(sts)
      this.status = this.sts === 0 ? null : parseInt(sts)
      this.page.currentPage = 1
      this.getDataList(this.page)
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val
    },
    orderStatus(val) {
      this.status = val
      this.sts = val
      this.getDataList()
    },
    /**
     * 根据选项设置时间
     * 1:今天 2:昨天 3: 近七天 4:近30天 5:近60天
     */
    setDateRange(val) {
      var startDay = null
      var endDay = null
      if (val === 1) {
        startDay = 0
        endDay = 0
      } else if (val === 2) {
        startDay = -1
        endDay = -1
      } else if (val === 3) {
        startDay = -7
        endDay = -1
      } else if (val === 4) {
        startDay = -30
        endDay = -1
      } else {
        return
      }
      // 开始时间
      let startTime = moment().add(startDay, 'days').startOf('days').format('LL')
      // 结束时间
      let endTime = moment().add(endDay, 'days').endOf('days').format('LL')
      this.dateRange = [startTime, endTime]
    },
    // 新增 / 修改
    addOrUpdateHandle(val) {
      // this.addOrUpdateVisible = true
      // this.$nextTick(() => {
      //   this.$refs.addOrUpdate.init(val)
      // })
      this.$router.push({
        path: '/order-orderInfo',
        query: {
          orderNumber: val
        }
      })
    },
    // 前往消息盒子
    toImbox(order) {
      if (isAuth('message:message:view')) {
        window.open(location.href.split('#')[0] + '#/imBox?userId=' + order.userId + '&orderNumber=' + order.orderNumber, 'view_window')
      } else {
        this.$message.info(this.$i18n.t('home.authTip'))
      }
    },
    // 退款路由跳转
    refundRoute(val) {
      // 修改缓存页面
      this.$store.commit('router/updateIncludePageList', 'order-orderRefund')
      this.$store.commit('router/pushHisPageList', 'order-orderRefund')
      this.$router.push({
        path: '/order-orderRefund',
        query: { orderNumber: val }
      })
    },
    // 删除
    deleteHandle(id) {
      var ids = id ? [id] : this.dataListSelections.map(item => {
        return item.orderId
      })
      this.$confirm(this.$i18n.t('remindPop.makeSure') + ' ' + `[${id ? this.$i18n.t('remindPop.delete') : this.$i18n.t('remindPop.batchDeletion')}]` + ' ' + this.$i18n.t('remindPop.operation') + '?', this.$i18n.t('remindPop.remind'), {
        confirmButtonText: this.$i18n.t('remindPop.confirm'),
        cancelButtonText: this.$i18n.t('remindPop.cancel'),
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl(`/prod/spec/${ids}`),
          method: 'delete',
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          this.$message({
            message: this.$i18n.t('remindPop.succeeded'),
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.getDataList(this.page)
            }
          })
        })
      }).catch(() => { })
    },
    showConsignmentInfo() {
      this.consignmentInfoVisible = true
      this.$nextTick(() => {
        this.$refs.consignmentInfo.init()
      })
    },
    // 清空按钮
    clear() {
      this.dataForm = {}
      this.dateRange = []
      this.status = null
    },
    // 搜索查询
    searchChange(newData = false) {
      this.page.currentPage = 1
      this.getDataList(this.page, null, 0, newData)
    },
    // 订单导出
    getSoldExcel() {
      if (!this.dateRange || this.dateRange.length < 2) {
        this.$message.error(this.$i18n.t('order.pleExpOrderFirst'))
        return
      }
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
          text: this.$i18n.t('shop.exportIng')
        })
        this.$http({
          url: this.$http.adornUrl('/platform/order/soldExcel'),
          method: 'get',
          params: this.$http.adornParams({
            'orderNumber': this.dataForm.orderNumber,
            // 'prodName': this.dataForm.prodName,
            'shopName': this.dataForm.shopName,
            'orderType': this.dataForm.orderType,
            'payType': this.dataForm.payType,
            'receiver': this.dataForm.receiver,
            'mobile': this.dataForm.mobile,
            'lang': this.lang === 'en' ? 1 : 0,
            'status': this.status,
            'dvyType': this.dataForm.dvyType,
            'stationName': this.dataForm.stationName,
            'refundStatus': this.dataForm.refundStatus,
            'startTime': this.dateRange === null ? null : this.dateRange[0], // 开始时间
            'endTime': this.dateRange === null ? null : this.dateRange[1] // 结束时间
          }),
          responseType: 'blob' // 解决文件下载乱码问题
        }).then(({ data }) => {
          loading.close()
          var blob = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8' })
          const fileName = this.$i18n.t('order.orderInfCollationXls')
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
    // 订单按商品导出
    getSoldExcelByProd() {
      if (!this.dataForm1.prodId) {
        this.$message.error('请选择要导出的商品')
        return
      }
      const loading = this.$loading({
        lock: true,
        target: '.mod-order-order',
        customClass: 'export-load',
        background: 'transparent',
        text: this.$i18n.t('shop.exportIng')
      })
      this.$http({
        url: this.$http.adornUrl('/platform/order/prod/soldExcel'),
        method: 'get',
        params: this.$http.adornParams({
          'prodId': this.dataForm1.prodId,
        }),
        responseType: 'blob' // 解决文件下载乱码问题
      }).then(({ data }) => {
        loading.close()
        var blob = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8' })
        const fileName = '商品订单导出'
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
    },
    // 获取商品列表接口
    getProdList() {
      this.$http({
        url: this.$http.adornUrl('/platform/search/prod/page'),
        method: 'get',
        params: this.$http.adornParams({
          current: 1,
          size: 10000
        })
      }).then(({ data }) => {
        this.prodDataList = data.records
      })
    }
  },
  destroyed() {
    // 页面销毁时移除监听
    window.removeEventListener('scroll', this.handleScroll)
  }
}
</script>
<style lang="scss" scoped>
.mod-order-order {
  .search-bar {
    .input-row {
      .select-time-btn {
        margin-right: 20px;
        display: inline-block;
        color: #AAAAAA;
        font-size: 14px;
        cursor: pointer;

        &:last-child {
          margin-right: 0;
        }
      }

      .select-time-btn.is-active {
        color: #155BD4;
      }

    }
  }

  .main {
    .order-status-nav {
      position: relative;
      display: block;
      width: 100%;
      margin-bottom: 15px;
      // height: 40px;
      line-height: 40px;
      border-bottom: 1px solid #ddd;

      ul,
      li {
        list-style: none;
        padding: 0;
        margin: 0;
      }

      .nav-list {
        display: flex;
        flex-wrap: nowrap;
        flex: 0 0 auto;
      }

      .nav-item {
        min-width: 79px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;

        float: left;
        height: 40px;
        line-height: 40px;
        background: #f8f8f9;
        border: 1px solid #ddd;
        padding: 0 20px;
        margin: 0 -1px -1px 0;
        cursor: pointer;
      }

      .selected {
        background: #fff;
        border-bottom: 1px solid #fff;
      }
    }

    .status-nav-bottom {
      margin-bottom: 72px;
    }

    .tit {
      display: flex;
      align-items: center;
      margin-bottom: 15px;
      background: #F7F8FA;
      z-index: 11;
      height: 57px;
      font-weight: bold;

      .item {
        padding: 0 10px;
        width: 10%;
        text-align: center;
      }

      .product {
        width: 25%;
        margin-bottom: 15px
      }
    }

    .fixed-top {
      position: fixed;
      width: calc(100% - 310px);
      min-width: 890px;
      top: 90px;
    }

    .fixed-top-fold {
      position: fixed;
      width: calc(100% - 160px);
      min-width: 1040px;
      top: 90px;
    }

    .prod {
      margin-bottom: 15px;
      font-size: 14px;

      .prod-tit {
        padding: 10px;
        background: #F7F8FA;
        height: 49px;
        display: flex;
        align-items: center;
        border-left: 1px solid #EBEDF0;
        border-top: 1px solid #EBEDF0;
        border-right: 1px solid #EBEDF0;

        .order-number {
          color: #333333;
          font-size: 14px
        }

        .order-time {
          color: #999999;
          font-size: 14px
        }

        span {
          margin-right: 15px;
        }
      }

      .prod-cont {
        display: flex;
        border: 1px solid #EBEDF0;
        color: #495060;
        font-size: 14px;

        .item {
          display: flex;
          display: -webkit-flex;
          align-items: center !important;
          padding: 10px;
          justify-content: center !important;
          height: 100%;
          border-right: 1px solid #eee;

          .totalprice {
            color: #ff4141;
            margin-bottom: 10px;
          }

          .operate {
            color: #2d8cf0;
            height: auto;

            .operate-btn {
              height: auto;
              margin: 0 !important;
              display: block !important;
            }

            .default-btn+.default-btn {
              height: auto;
              display: block;
              margin-left: 0;
            }
          }

          .buyer-info {
            .buyer-name {
              margin-bottom: 10px;
            }
          }

          span {
            display: block;
            margin-bottom: 0 !important;
          }

          span+span {
            margin-top: 10px;
          }
        }

        .prod-item {
          padding: 0;
          display: flex;
          flex-direction: column;
          height: 100%;
          border-right: 1px solid #eee;

          .items.name {
            width: 100%;
            display: flex;
            align-items: center;
            border-bottom: 1px solid #EBEDF0;
            padding: 10px;

            &:last-child {
              border-bottom: none;
            }

            .order-prod-item-info {
              width: 72%;
              display: flex;
              flex-direction: column;

              .info {
                display: flex;
                align-items: center;

                .prod-image {
                  // width: 80px;
                  width: 17.5%;
                  min-height: 80px;
                  height: auto;
                  margin-right: 20px;
                  line-height: 80px;

                  img {
                    width: 100%;
                    min-height: 100%;
                    object-fit: contain;
                  }
                }

                .prod-name {
                  flex: 1;
                  margin-left: auto;
                  max-width: 78% !important;

                  .prod-con {
                    text-align: left;
                    width: 85% !important;
                    display: block;
                    padding: 0 !important;

                    .prod-name-txt {
                      padding-right: 10px;
                      box-sizing: border-box;
                      display: -webkit-box;
                      word-break: break-word;
                      -webkit-box-orient: vertical;
                      -webkit-line-clamp: 2;
                      overflow: hidden;
                    }

                    .prod-name-sku {
                      color: #999;
                    }

                    .order-status {
                      display: inline-block;
                      margin-top: 15px;
                      margin-right: 10px;
                      padding: 2px 4px;
                      border: 1px solid #e43130;
                      border-radius: 2px;
                      color: #e43130;
                    }
                  }
                }
              }

              // 赠品盒子
              .order-prod-item-give-con {
                width: 100%;
                padding: 10px 50px 0 0;
                box-sizing: border-box;

                .giveaway-item {
                  display: flex;
                  margin-bottom: 10px;

                  &:last-child {
                    margin-bottom: 0;
                  }
                }

                .giveaway-name {
                  max-width: 70%;
                  overflow: hidden;
                  white-space: nowrap; //禁止换行
                  text-overflow: ellipsis;
                }

                .giveaway-sku-name {
                  margin-left: 10px;
                }

                .giveaway-item-sku-count {
                  width: auto;
                  margin-left: 10px;
                  color: #999;
                  overflow: initial;
                }
              }
            }

            .prod-price {
              width: 28%;
              display: flex;
              justify-content: flex-start;
              flex-direction: column;
              overflow: hidden;
              position: relative;
              right: 0;

              span {
                display: block;
                text-align: center;
                word-break: keep-all;
              }
            }
          }
        }
      }
    }

    .empty {
      display: block;
      height: 200px;
      line-height: 200px;
      text-align: center;
      color: #aaa;
    }

    .transaction-price {
      text-align: center;
    }
  }

  ::v-deep .export-load {
    top: -50% !important;
  }

}

div ::v-deep .el-form-item--small .el-form-item__content {
  display: flex;
  flex-wrap: wrap;
}
</style>
<style>
/* chrome */
.mod-order-order .search-bar .input-row .myinput-appearance>input::-webkit-outer-spin-button,
.mod-order-order .search-bar .input-row .myinput-appearance>input::-webkit-inner-spin-button {
  -webkit-appearance: none;
}

/* 火狐浏览器 */
.mod-order-order .search-bar .input-row .myinput-appearance>input[type="number"] {
  -moz-appearance: textfield;
}
</style>

<style scoped lang="scss">
::v-deep .search-bar .input-row .el-form-item__content .el-input {
  width: 100%;
}
</style>
