<template>
  <div style="width: 80%; margin: 2px auto">
    <div style="margin-top: 20px">
      <el-input style="width: 350px; margin-right: 10px" v-model="data.name" clearable @clear="reset" placeholder="请输入名称查询"></el-input>
      <el-button type="primary" plain @click="load">查询</el-button>
    </div>
    <div style="margin-top: 10px">
      <el-row :gutter="10">
       <el-col :span="5" v-for="item in data.goodsData" style="margin-bottom: 10px">
         <div v-if="item.maxTime > 0">
           <img :src="item.img" alt="" style="width: 100%; height: 235px; border: 1px solid #cccccc; border-radius: 10px">
           <div class="overflow" style="font-size: 17px; font-weight: bold; color: #333333; margin-top: 10px">{{ item.name }}</div>
           <div style="margin-top: 5px; color: red">秒杀倒计时：{{ item.hour }}:{{ item.minutes }}:{{ item.seconds }}:{{ item.flashDown }}</div>
           <div style="display: flex; align-items: center; color: red; margin-top: 5px; font-weight: bold; font-size: 15px">
             <div style="flex: 1">秒杀价格：￥{{ item.flashPrice }}</div>
             <div style="width: 80px">
               <el-button type="danger" @click="navTo('/front/goodsDetail?id=' + item.id)">去秒杀</el-button>
             </div>
           </div>
         </div>
       </el-col>
      </el-row>
    </div>
    <div v-if="data.total">
      <el-pagination @current-change="load" background layout="total, prev, pager, next"
                     :total="data.total" v-model:current-page="data.pageNum" v-model:page-size="data.pageSize" />
    </div>
  </div>
</template>

<script setup>
import {reactive} from "vue";
import request from "@/utils/request.js";
import {ElMessage} from "element-plus";
import router from "@/router/index.js";

const data = reactive({
  name: null,
  pageNum: 1,
  pageSize: 10,
  total: 0,
  goodsData: [],
})

const load = () => {
  request.get('/goods/selectPage', {
    params: {
      pageNum: data.pageNum,
      pageSize: data.pageSize,
      hasFlash: true,
      hasGroup: false,
      name: data.name,
    }
  }).then(res => {
    data.goodsData = res.data.list
    data.total = res.data.total
    data.goodsData.forEach(item => {
      item.flashDown = 9
    })
    setInterval(flashDown, 100)
  })
}

const reset = () => {
  data.name = null
  load()
}
const navTo = (url) => {
  location.href = url
}

const flashDown = () => {
  data.goodsData?.forEach(item => {
    let maxTime = item.maxTime
    if (maxTime > 0) {
      let remain = Math.floor(maxTime % 3600)
      item.hour = Math.floor(maxTime / 3600)
      item.minutes = Math.floor(remain / 60)
      item.seconds = Math.floor(remain % 60)
      if (item.flashDown === 0) {
        item.maxTime--
        item.flashDown = 9
      } else {
        item.flashDown--
      }

    }
  })
}

load()


</script>

<style scope>
.overflow {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis
}
.ellipsis2 {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}
.el-col-5 {
  width: 20%;
  max-width: 20%;
  padding: 10px 10px;
}
</style>