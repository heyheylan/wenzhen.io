<script setup lang="ts">
import { OrderType } from '@/enums'
import { getMedicalOrderDetailAPI } from '@/apis/order'
import type { OrderDetail } from '@/types/order'
import { onMounted, ref } from 'vue'
import { useRoute } from 'vue-router'

//药品处方信息
const order = ref<OrderDetail>()
const route = useRoute()
onMounted(async ()=>{
  const res = await getMedicalOrderDetailAPI(route.query.orderId as string)
  console.log(res)
  order.value = res.data
})

</script>

<template>
  <div class="order-pay-result-page">
    <cp-nav-bar :LeftTo="`/room?orderId=${order?.roomId}`" :arrow="true" title="药品支付结果" />
    <div class="result error" v-if="order?.status === OrderType.ConsultPay">
      <van-icon name="clear" />
      <p class="price">￥ {{ order?.actualPayment }}</p>
      <p class="status">支付失败</p>
      <p class="tip">订单支付失败，可以点击查看订单继续支付，如有疑问联系客服~</p>
      <div class="btn">
        <van-button type="primary" :to="`/order/${order?.id}`">查看订单</van-button>
        <van-button :to="`/room?orderId=${order?.roomId}`">返回诊室</van-button>
      </div>
    </div>
    <div class="result" v-else>
      <van-icon name="checked" />
      <p class="price">￥ {{ order?.actualPayment }}</p>
      <p class="status">支付成功</p>
      <p class="tip">订单支付成功，已通知药房尽快发出， 请耐心等待~</p>
      <div class="btn">
        <van-button type="primary" :to="`/order/${order?.id}`">查看订单</van-button>
        <van-button :to="`/room?orderId=${order?.roomId}`">返回诊室</van-button>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.order-pay-result-page {
  padding-top: 46px;
}
.result {
  display: flex;
  flex-direction: column;
  align-items: center;
  .van-icon {
    font-size: 75px;
    color: var(--cp-primary);
    margin-top: 60px;
  }
  .price {
    font-size: 22px;
    margin-top: 10px;
  }
  .status {
    color: var(--cp-text3);
  }
  .tip {
    color: var(--cp-tip);
    width: 240px;
    text-align: center;
    margin-top: 20px;
  }
  .btn {
    margin-top: 60px;
    .van-button--primary {
      margin-right: 20px;
    }
  }
  &.error {
    .van-icon {
      color: var(--cp-price);
    }
  }
}
</style>
