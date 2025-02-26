<script setup lang="ts">
import type { ConsultOrderItem } from '@/types/consult'
import { OrderType } from '@/enums'
import {computed,ref,defineEmits} from 'vue'
import {cancelOrderAPI,deleteOrderAPI} from "@/apis/consult";
import {showSuccessToast, showFailToast, showConfirmDialog} from 'vant'
import {useShowPrescription} from "@/composables/ShowPrescription";
import router from "@/router";
import {useCancelOrder} from "@/composables/UseCancelOrder";
import {UseDeleteConsultOrder} from "@/composables/UseDeleteConsultOrder";


//定义删除emit
const emit = defineEmits<{
  (e:'on-delete',id:string):void
}>()

const {deleteConsultOrder,deleteLoading} = UseDeleteConsultOrder(()=>{
  emit('on-delete',props.item.id)
})


//展示处方函数
const {onShowPrescription} = useShowPrescription()
//取消订单函数
const {loading,cancelOrder} = useCancelOrder()

//接收当前item项
const props = defineProps<{ item: ConsultOrderItem }>()
// console.log(props.item.status)
const showPopover = ref(false)
const actions = computed(()=>[
  {text:'查看处方',disabled:!props.item.prescriptionId},
  {text:'删除订单'}
])
//选项
const onSelect = (action: { text: string }, i: number)=>{
  //查看处方
  if(i===0){
    onShowPrescription(props.item.prescriptionId)
  }
  if(i===1){
    //删除
    deleteConsultOrder(props.item)
  }

}
//取消订单
// const loading = ref(false)
// const cancelOrder = async (item:ConsultOrderItem)=>{
//   try{
//     loading.value = true
//     await cancelOrderAPI(item.id)
//     //改状态为取消问诊
//     item.status=OrderType.ConsultCancel
//     item.statusValue = '已取消'
//     showSuccessToast('取消成功')
//   }catch(e){
//     showFailToast('取消失败')
//   }
//   loading.value = false
// }

//删除订单

// const deleteLoading = ref(false)
// const deleteConsultOrder = async (item:ConsultOrderItem) =>{
//   throw showConfirmDialog({
//     title: '温馨提示',
//     message: '是否确定删除订单？',
//     closeOnPopstate: false
//   }).then(async ()=>{
//     try{
//       deleteLoading.value = true
//       await deleteOrderAPI(item.id)
//       showSuccessToast('删除成功')
//       emit('on-delete',item.id)
//     }catch(e){
//       showFailToast('删除失败')
//     }finally{
//       deleteLoading.value = false
//     }
//   })
// }

</script>

<template>
  <div class="consult-item">
    <div class="head van-hairline--bottom">
      <img class="img" src="../../../assets/avatar-doctor.svg" />
      <p>{{ item.docInfo?.name || '暂未分配医生' }}</p>
      <span
          :class="{
          orange: item.status === OrderType.ConsultPay,
          green: item.status === OrderType.ConsultChat
        }"
      >{{ item.statusValue }}</span>
    </div>
    <div class="body" @click="router.push(`/user/consult/${item.id}`)">
      <div class="body-row">
        <div class="body-label">病情描述</div>
        <div class="body-value">{{ item.illnessDesc }}</div>
      </div>
      <div class="body-row">
        <div class="body-label">价格</div>
        <div class="body-value">¥ {{ item.payment.toFixed(2) }}</div>
      </div>
      <div class="body-row">
        <div class="body-label">创建时间</div>
        <div class="body-value tip">{{ item.createTime }}</div>
      </div>
    </div>
    <!--待支付-->
    <div class="foot" v-if="item.status === OrderType.ConsultPay">
      <van-button class="gray" plain size="small" round :loading="loading" @click="cancelOrder(item)">取消问诊</van-button>
      <van-button type="primary" plain size="small" round :to="`/user/consult/${item.id}`">
        去支付
      </van-button>
    </div>
    <!--带接诊-->
    <div class="foot" v-if="item.status === OrderType.ConsultWait">
      <van-button class="gray" plain size="small" round :loading="loading" @click="cancelOrder(item)">取消问诊</van-button>
      <van-button type="primary" plain size="small" round :to="`/room?orderId=${item.id}`">
        继续沟通
      </van-button>
    </div>
    <!--咨询中-->
    <div class="foot" v-if="item.status === OrderType.ConsultChat">
      <van-button @click="onShowPrescription(item.prescriptionId)" v-if="item.prescriptionId" class="gray" plain size="small" round>
        查看处方
      </van-button>
      <van-button type="primary" plain size="small" round :to="`/room?orderId=${item.id}`">
        继续沟通
      </van-button>
    </div>
    <!--已完成-->
    <div class="foot" v-if="item.status === OrderType.ConsultComplete">
      <div class="more">
        <!--&lt;!&ndash;@select:点击选项时触发&ndash;&gt;-->
        <!--<van-popover-->
        <!--    placement="top-start"-->
        <!--    v-model:show="showPopover"-->
        <!--    :actions="actions"-->
        <!--    @select="onSelect"-->
        <!--&gt;-->
        <!--  <template #reference> 更多 </template>-->
        <!--</van-popover>-->
        <cp-consult-more
            :disabled="!item.prescriptionId"
            @on-delete="deleteConsultOrder(item)"
            @on-preview="onShowPrescription(item.prescriptionId)"
        ></cp-consult-more>
      </div>
      <van-button class="gray" plain size="small" round :to="`/room?orderId=${item.id}`">
        问诊记录
      </van-button>
      <van-button v-if="!item.evaluateId" type="primary" plain size="small" round>
        去评价
      </van-button>
      <van-button v-else class="gray" plain size="small" round> 查看评价 </van-button>
    </div>
    <!--已取消-->
    <div class="foot" v-if="item.status === OrderType.ConsultCancel">
      <van-button class="gray" plain size="small" round
                  :loading="deleteLoading"
                  @click="deleteConsultOrder(item)">删除订单</van-button>
      <van-button type="primary" plain size="small" round to="/">咨询其他医生</van-button>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.consult-item {
  border-radius: 4px;
  box-shadow: 0px 0px 22px 0px rgba(245, 245, 245, 0.1);
  background-color: #fff;
  margin-bottom: 10px;
  .head {
    display: flex;
    align-items: center;
    height: 50px;
    padding: 0 15px;
    .img {
      width: 20px;
      height: 20px;
    }
    > p {
      flex: 1;
      font-size: 15px;
      padding-left: 10px;
    }
    > span {
      color: var(--cp-tag);
      &.orange {
        color: #f2994a;
      }
      &.green {
        color: var(--cp-primary);
      }
    }
  }
  .body {
    padding: 15px 15px 8px 15px;
    .body-row {
      display: flex;
      margin-bottom: 7px;
    }
    .body-label {
      width: 62px;
      font-size: 13px;
      color: var(--cp-tip);
    }
    .body-value {
      width: 250px;
      &.tip {
        color: var(--cp-tip);
      }
    }
  }
  .foot {
    padding: 0 15px 15px 15px;
    display: flex;
    justify-content: flex-end;
    align-items: center;
    .van-button {
      margin-left: 10px;
      padding: 0 16px;
      &.gray {
        color: var(--cp-text3);
        background-color: var(--cp-bg);
      }
    }
    .more {
      color: var(--cp-tag);
      flex: 1;
      font-size: 13px;
    }
  }
}
</style>