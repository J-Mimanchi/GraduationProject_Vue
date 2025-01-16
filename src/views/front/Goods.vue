<template>
  <div style="width: 80%; margin: 2px auto">
    <div style="margin-top: 20px">
      <el-input style="width: 350px; margin-right: 10px" v-model="data.name" clearable @clear="reset" placeholder="请输入名称查询"></el-input>
      <el-button type="primary" plain @click="load">查询</el-button>
    </div>
    <div style="margin-top: 10px">
      <el-tag type="primary" size="large" v-for="item in data.categoryData" style="margin-right: 10px; cursor: pointer" @click="changeCategory(item.id)">{{ item.name }}</el-tag>
    </div>
    <div style="margin-top: 10px">
      <el-row :gutter="10">
       <el-col :span="5" v-for="item in data.goodsData" style="margin-bottom: 10px">
         <img :src="item.img" alt="" style="width: 100%; height: 235px; border: 1px solid #cccccc; border-radius: 10px">
         <div class="overflow" style="font-size: 17px; font-weight: bold; color: #333333; margin-top: 10px">{{ item.name }}</div>
         <div style="display: flex; align-items: center; color: red; margin-top: 5px; font-weight: bold; font-size: 15px">
           <div style="flex: 1">商品价格：￥{{ item.originPrice }}</div>
           <div style="width: 80px">
             <el-button type="warning" @click="navTo('/front/goodsDetail?id=' + item.id)">去购买</el-button>
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
  name: router.currentRoute.value.query.name,
  pageNum: 1,
  pageSize: 10,
  total: 0,
  goodsData: [],
  categoryData: [],
  categoryId: null,
})

const load = () => {
  request.get('/goods/selectPage', {
    params: {
      pageNum: data.pageNum,
      pageSize: data.pageSize,
      hasFlash: false,
      hasGroup: false,
      name: data.name,
      categoryId: data.categoryId
    }
  }).then(res => {
    data.goodsData = res.data.list
    data.total = res.data.total
  })
}

const loadCategory = () => {
  request.get('/category/selectAll').then(res => {
    if (res.code === '200') {
      data.categoryData = res.data
    } else {
      ElMessage.error(res.msg)
    }
  })
}
const changeCategory = (id) => {
  data.categoryId = id
  load()
}

const reset = () => {
  data.name = null
  load()
}
const navTo = (url) => {
  location.href = url
}

load()
loadCategory()


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