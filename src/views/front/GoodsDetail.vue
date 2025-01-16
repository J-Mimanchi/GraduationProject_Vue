<template>
  <div style="width: 60%; margin: 20px auto">
    <el-row :gutter="15">
     <el-col :span="7">
       <img :src="data.goodsData.img" alt="" style="width: 100%; height: 250px; border-radius: 10px; border: 1px solid #cccccc">
     </el-col>
     <el-col :span="17">
       <div class="overflow" style="font-size: 20px; font-weight: bold; margin-bottom: 10px">{{ data.goodsData.name }}</div>
       <div v-if="data.goodsData.hasFlash" style="margin: 10px 0 20px 0; background-image: linear-gradient(to right, #f83939, #f52593); padding: 10px; border-radius: 5px">
         <div style="font-size: 22px; color: white; font-weight: bold">秒杀价：￥{{ data.goodsData.flashPrice }}</div>
         <div style="margin-top: 5px; color: white">原价：<del>￥{{ data.goodsData.originPrice }}</del></div>
       </div>
       <div v-else-if="data.goodsData.hasGroup" style="margin: 10px 0 20px 0; background-image: linear-gradient(to right, #faa303, #ef9d84); padding: 10px; border-radius: 5px">
         <div style="font-size: 22px; color: white; font-weight: bold">团购价：￥{{ data.goodsData.groupPrice }}</div>
         <div style="margin-top: 5px; color: white">原价：<del>￥{{ data.goodsData.originPrice }}</del></div>
       </div>
       <div v-else style="color: red; font-size: 18px; font-weight: bold; margin: 20px 0">商品价格：￥{{ data.goodsData.originPrice }}</div>
       <div style="font-size: 15px">商品分类：{{data.goodsData.categoryName}}</div>
       <div style="margin-top: 10px; font-size: 15px" v-if="data.goodsData.hasFlash || data.goodsData.hasGroup">
         <span>购买数量：</span>
         <el-input-number v-model="data.num" :min="1" :max="data.goodsData.store"/>
         <span style="margin-left: 20px">库存 {{ data.goodsData.store }}</span>
       </div>
       <div style="margin-top: 20px; font-size: 15px" v-else>
         <span>购买数量：</span>
         <el-input-number v-model="data.num" :min="1" :max="data.goodsData.store" @change="initNum"/>
         <span style="margin-left: 20px">库存 {{ data.goodsData.store }}</span>
       </div>
       <div style="margin-top: 20px" v-if="data.goodsData.hasFlash || data.goodsData.hasGroup">
         <el-button type="danger" style="padding: 20px 30px" v-if="data.goodsData.hasFlash" @click="createFlashOrder">去秒杀</el-button>
         <el-button type="warning" style="padding: 20px 30px" v-if="data.goodsData.hasGroup" @click="createGroupOrder">去团购</el-button>
       </div>
       <div style="margin-top: 35px" v-else>
         <el-button type="primary" style="padding: 20px 30px" @click="createOrdinaryOrder">去购买</el-button>
       </div>
     </el-col>
    </el-row>
    <div style="margin-top: 20px">
      <el-tabs type="border-card">
        <el-tab-pane label="商品详情">
          <div v-html="data.goodsData.content"></div>
        </el-tab-pane>
        <el-tab-pane label="商品评价">
          <div v-if="!data.commentData.length" style="color: #666666">该商品暂无评价</div>
          <div v-else style="background-color: white; border-radius: 10px; padding: 10px">
            <div>
              <div style="display: flex; margin-top: 30px" v-for="item in data.commentData">
                <img :src="item.userAvatar" alt="" style="width: 50px; height: 50px; border-radius: 50%">
                <div style="flex: 1; margin-left: 15px">
                  <div style="color: #61666d; font-weight: 550">{{ item.userName }}</div>
                  <div style="margin-top: 10px; line-height: 25px">{{ item.content }}</div>
                  <div style="display: flex; margin-top: 10px; align-items: center">
                    <div><el-rate v-model="item.score" disabled/></div>
                    <div style="color: #999999; margin-left: 20px">{{ item.createTime }}</div>
                  </div>
                  <div style="display: flex; margin-top: 15px" v-if="item.reply">
                    <img src="@/assets/imgs/404.jpg" alt="" style="width: 50px; height: 50px; border-radius: 50%">
                    <div style="flex: 1; margin-left: 15px">
                      <div style="color: #61666d; font-weight: 550">商家 回复：{{ item.userName }}</div>
                      <div style="margin-top: 10px; line-height: 25px">{{ item.reply }}</div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </el-tab-pane>
      </el-tabs>
    </div>
  </div>
</template>

<script setup>
import {reactive, onMounted} from "vue";
import request from "@/utils/request.js";
import {ElMessage} from "element-plus";
import router from "@/router/index.js";

const data = reactive({
  num: 1,
  goodsId: router.currentRoute.value.query.id,
  orderId: router.currentRoute.value.query.orderId,
  goodsData: {},
  commentData: []
})

const loadGoods = () => {
  data.goodsId = router.currentRoute.value.query.id
  request.get('/goods/selectById/' + data.goodsId).then(res => {
    if (res.code === '200') {
      data.goodsData = res.data
    } else {
      ElMessage.error(res.msg)
    }
  })
}

const initNum = (num) => {
  data.num = num
}

const createOrdinaryOrder = () => {
  let orderData = {
    goodsId: data.goodsId,
    num: data.num,
    type: 'COMMON'
  }
  request.post('/orders/add', orderData).then(res => {
    if (res.code === '200') {
      // 创建订单成功后，会返回这个订单信息，然后我们跳转到确认订单页
      location.href = '/front/ordersDetail?id=' + res.data.id
    }
  })
}
const createFlashOrder = () => {
  let orderData = {
    goodsId: data.goodsId,
    num: data.num,
    type: 'FLASH'
  }
  request.post('/orders/addFlashOrder', orderData).then(res => {
    if (res.code === '200') {
      // 创建订单成功后，会返回这个订单信息，然后我们跳转到确认订单页
      location.href = '/front/ordersDetail?id=' + res.data.id
    }
  })
}
const createGroupOrder = () => {
  let orderData = {
    goodsId: data.goodsId,
    num: data.num,
    type: 'GROUP',
    groupOrderId: data.orderId
  }
  request.post('/orders/add', orderData).then(res => {
    if (res.code === '200') {
      // 创建订单成功后，会返回这个订单信息，然后我们直接走支付
      window.open('/pay?orderNo=' + res.data.orderNo)
    }
  })
}

const loadComment = () => {
  request.get('/comment/selectAll', {
    params: {
      goodsId: data.goodsId
    }
  }).then(res => {
    if (res.code === '200') {
      data.commentData = res.data
    } else {
      ElMessage.error(res.msg)
    }
  })
}

loadGoods()
loadComment()



</script>

<style scope>
.overflow {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis
}
</style>