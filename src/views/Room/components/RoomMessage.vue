<script setup lang="ts">
import {useShowPrescription} from "@/composables/ShowPrescription";
import {IllnessTime, MsgType} from '@/enums'
import {flagOptions, timeOptions} from '@/apis/consult'
import type {Message, Prescription} from "@/types/room";
import {showFailToast, showImagePreview, showToast} from 'vant'
import type {Image} from "@/types/consult";
import type {ConsultOrderItem} from '@/types/consult'
import {getConsultOrderDetailAPI} from '@/apis/consult'
import {onMounted, ref} from "vue";
import {useRoute} from "vue-router";
import {useUserStore} from '@/stores'
import dayjs from "dayjs";
import {useRouter} from 'vue-router'
import {PrescriptionStatus} from '@/enums'
import {showSuccessToast} from "vant"
import router from "@/router";
import EvaluateCard from '@/views/Room/components/evaluateCard.vue'


const store = useUserStore()

//时间格式化处理
const formatTime = (time: string) => dayjs(time).format('HH:mm')

const routes = useRoute()
const consult = ref<ConsultOrderItem>()
onMounted(async () => {
  const res = await getConsultOrderDetailAPI(routes.query.orderId as string)
  consult.value = res.data
})

//接收聊天数据
const props = defineProps<{ item: Message }>()
//常量聊天消息处理
const getIllnessTimeText = (time: IllnessTime) => {
  return timeOptions.find((item) => item.value === time)?.label
}
const getConsultFlagText = (flag: 0 | 1) => {
  return flagOptions.find((item) => item.value === flag)?.label
}

//图片处理
// 预览图片
const showPreviewImage = (img?: Image[]) => {
  if (img && img.length) {
    showImagePreview(img.map((item) => item.url))
  } else {
    showFailToast('暂无图片')
  }
}


const {onShowPrescription} = useShowPrescription()

const orderID = props.item.msg.prescription?.orderId
//购买药品
const buy = (pre?: Prescription) => {
  if (pre) {
    if (pre.status === PrescriptionStatus.Invalid) {
      showFailToast('该处方已过期，无法购买')
    }
    if (pre.status === PrescriptionStatus.NotPayment && !pre.orderId) {
      return router.push(`/order/pay?id=${pre.id}`)
    }
    // orderID.value = pre.orderId
    // console.log('orderID', orderID.value)
    //有药物订单了，但是还没支付
    router.push(`/order/${pre.orderId}`)

  }
}
</script>


<template>
  <!-- 患者卡片 -->
  <div class="msg msg-illness" v-if="item.msgType === MsgType.CardPat">
    <div class="patient van-hairline--bottom" v-if="item.msg.consultRecord">
      <p>
        {{ item.msg.consultRecord.patientInfo.name }}
        {{ item.msg.consultRecord.patientInfo.genderValue }}
        {{ item.msg.consultRecord.patientInfo.age }}岁
      </p>
      <p> {{ getIllnessTimeText(item.msg.consultRecord.illnessTime) }} |
        {{ getConsultFlagText(item.msg.consultRecord.consultFlag) }}</p>
    </div>
    <van-row>
      <van-col span="6">病情描述</van-col>
      <van-col span="18">{{ item.msg.consultRecord?.illnessDesc }}</van-col>
      <van-col span="6">图片</van-col>
      <van-col span="18"
               @click="showPreviewImage(item.msg.consultRecord?.pictures)">点击查看
      </van-col>
    </van-row>
  </div>
  <!--通知-通用 -->
  <div class="msg msg-tip" v-if="item.msgType === MsgType.Notify">
    <div class="content">
      <span>{{ item.msg.content }}</span>
    </div>
  </div>

  <!-- 通知-温馨提示 -->
  <div class="msg msg-tip" v-if="item.msgType === MsgType.NotifyTip">
    <div class="content">
      <span class="green">温馨提示：</span>
      <span>{{ item.msg.content }}</span>
    </div>
  </div>

  <!--通知-结束 -->
  <div class="msg msg-tip msg-tip-cancel" v-if="item.msgType === MsgType.NotifyCancel">
    <div class="content">
      <span>{{ item.msg.content }}</span>
    </div>
  </div>
  <!-- 发送文字 -->
  <div class="msg msg-to" v-if="item.msgType === MsgType.MsgText && store.user?.id === item.from">
    <div class="content">
      <div class="time">{{ formatTime(item.createTime) }}</div>
      <div class="pao">{{ item.msg.content }}</div>
    </div>
    <van-image :src="store.user?.avatar"/>
  </div>
  <!-- 发送图片 -->
  <div class="msg msg-to" v-if="item.msgType === MsgType.MsgImage && store.user?.id === item.from">
    <div class="content">
      <div class="time">{{ formatTime(item.createTime) }}</div>
      <van-image
          fit="contain"
          :src="item.msg.picture?.url"
      />
    </div>
    <van-image :src="store.user?.avatar"/>
  </div>
  <!-- 接收文字 -->
  <div class="msg msg-from" v-if="item.msgType === MsgType.MsgText && store.user?.id !== item.from">
    <van-image :src="item.fromAvatar"/>
    <div class="content">
      <div class="time">{{ formatTime(item.createTime) }}</div>
      <div class="pao">{{ item.msg.content }}</div>
    </div>
  </div>
  <!-- 接收图片 -->
  <div class="msg msg-from" v-if="item.msgType === MsgType.MsgImage && store.user?.id !== item.from">
    <van-image :src="item.fromAvatar"/>
    <div class="content">
      <div class="time">{{ formatTime(item.createTime) }}</div>
      <van-image fit="contain" :src="item.msg.picture?.url"/>
    </div>
  </div>
  <!-- 处方卡片 -->
  <div class="msg msg-recipe" v-if="item.msgType === MsgType.CardPre">
    <!--如果有处方信息-->
    <div class="content" v-if="item.msg.prescription">
      <div class="head van-hairline--bottom">
        <div class="head-tit">
          <h3>电子处方</h3>
          <p @click="onShowPrescription(item.msg.prescription?.id)">原始处方
            <van-icon name="arrow"></van-icon>
          </p>
        </div>
        <p>{{ item.msg.prescription.name }}
          {{ item.msg.prescription.genderValue }}
          {{ item.msg.prescription.age }}岁
          {{ item.msg.prescription.diagnosis }}</p>
        <p>开方时间：{{ item.msg.prescription.createTime }}</p>
      </div>
      <div class="body">
        <div class="body-item" v-for="med in item.msg.prescription.medicines" :key="med.id">
          <div class="durg">
            <p>{{ med.name }} {{ med.specs }}</p>
            <p>{{ med.name }} {{ med.specs }}</p>
          </div>
          <div class="num">x{{ med.quantity }}</div>
        </div>
      </div>
      <div class="foot" @click="buy(item.msg.prescription)"><span>{{ orderID ? '已购买' : '购买药品' }}</span></div>
    </div>
  </div>
  <!-- 评价卡片，后期实现 -->
  <div
      class="msg msg-comment"
      v-if="item.msgType === MsgType.CardEva || item.msgType === MsgType.CardEvaForm">
    <evaluate-card :evaluateDoc="item.msg.evaluateDoc" />
  </div>

  <!--结束问诊消息-->
  <div class="msg msg-tip msg-tip-cancel" v-if="item.msgType === MsgType.NotifyCancel">
    <div class="content">
      <span>{{ item.msg.content }}</span>
    </div>
  </div>

</template>


<style lang="scss" scoped>
@import '@/styles/room.scss';
</style>