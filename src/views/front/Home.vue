<template>
  <div style="width: 80%; margin: 2px auto">
    <!--轮播图-->
    <div class="carousel-wrapper">
      <el-carousel height="350px" :interval="5000" arrow="hover" indicator-position="outside">
        <el-carousel-item v-for="item in data.carouselData" :key="item">
          <div class="carousel-item">
            <img :src="item.img" alt="" class="carousel-image">
            <!-- 可以添加轮播图文字说明 -->
            <div class="carousel-caption">
              <h3>{{ item.title }}</h3>
              <p>{{ item.description }}</p>
            </div>
          </div>
        </el-carousel-item>
      </el-carousel>
    </div>
    <!--秒杀和团购-->
    <div style="margin-top: 20px; height: 350px">


      <el-row :gutter="10">
       <el-col :span="8">
         <div style="background-color: #faf5f5; height: 350px; border-radius: 10px; padding: 10px 15px;">
           <div style="height: 30px; line-height: 30px; font-size: 16px; font-weight: bold;">🔥火热秒杀</div>
           <div style="margin: 10px 0; height: 140px" v-for="item in data.flashData">
             <el-row :gutter="10">
              <el-col :span="9">
                <img :src="item.img" alt="" style="width: 135px; height: 135px; border-radius: 10px">
              </el-col>
              <el-col :span="15">
                <div class="ellipsis2" style="font-size: 17px; font-weight: bold">{{ item.name }}</div>
                <div style="margin-top: 5px; font-weight: bold; font-size: 16px; color: red">秒杀价：￥{{ item.flashPrice }}</div>
                <div style="margin-top: 5px; color: red">秒杀倒计时：{{ item.hour }}:{{ item.minutes }}:{{ item.seconds }}:{{ item.flashDown }}</div>
                <div style="margin-top: 8px">
                  <el-button type="danger" @click="navTo('/front/goodsDetail?id=' + item.id)">去秒杀</el-button>
                </div>
              </el-col>
             </el-row>
           </div>
         </div>
       </el-col>


        <el-col :span="16">
          <div class="group-carousel-container">
            <div class="group-header">
              <span class="group-title">🔥热门团购</span>
              <div class="group-indicators">
                <span v-for="page in data.pages" 
                      :key="page" 
                      :class="['indicator-dot', currentPage === page ? 'active' : '']">
                </span>
              </div>
            </div>
            <el-carousel
                height="300px"
                direction="vertical"
                :interval="4000"
                class="group-carousel"
                :pause-on-hover="false"
                indicator-position="none"
                @change="handleGroupChange"
            >
              <el-carousel-item v-for="it in data.pages" :key="it">
                <div class="group-item" v-for="item in data.groupData">
                  <img :src="item.goodsImg" alt="" class="group-item-image">
                  <div class="group-item-info">
                    <div class="group-item-title">{{ item.goodsName }}</div>
                    <div class="group-item-price">拼团价：￥{{ item.goodsPrice }}</div>
                  </div>
                  <div class="group-item-user">
                    <img :src="item.userAvatar" alt="" class="user-avatar">
                    <span class="user-name">{{ item.userName }}</span>
                    <span class="group-status">正在拼团中</span>
                  </div>
                  <div class="group-item-countdown">
                    倒计时：{{ item.hour }}:{{ item.minutes }}:{{ item.seconds }}:{{ item.groupDown }}
                  </div>
                  <el-button 
                    type="warning" 
                    class="join-group-btn"
                    @click="navTo('/front/GoodsDetail?id=' + item.goodsId + '&orderId=' + item.id)">
                    我要参团
                  </el-button>
                </div>
              </el-carousel-item>
            </el-carousel>
          </div>
       </el-col>
      </el-row>
    </div>

    <!--最新商品-->
    <div style="margin-top: 30px">
      <div style="display: flex">
        <div style="flex: 1; font-weight: bold; font-size: 18px">最新商品</div>
        <div style="width: 100px; text-align: right; color: #666666; cursor: pointer" @click="navTo('/front/goods')">查看更多</div>
      </div>
    </div>
    <div>
      <!--渲染商品-->
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
  </div>

</template>

<script setup>
import {reactive, ref} from "vue";
import request from "@/utils/request.js";
import {ElMessage} from "element-plus";

const data = reactive({
  carouselData: [],
  goodsData: [],
  flashData: [],
  groupData: [],
  pages: 1,
})

const currentPage = ref(1)

const loadCarouse = () => {
  request.get('/carousel/selectAll').then(res => {
    if (res.code === '200') {
      data.carouselData = res.data
    } else {
      ElMessage.error(res.msg)
    }
  })
}

const loadGoods = () => {
  request.get('/goods/selectPage', {
    params: {
      hasFlash: false,
      hasGroup: false,
      pageNum: 1,
      pageSize: 5,
    }
  }).then(res => {
    if (res.code === '200') {
      data.goodsData = res.data?.list
    } else {
      ElMessage.error(res.msg)
    }
  })
}

const loadFlash = () => {
  request.get('/goods/selectFlash', {
    params: {
      hasFlash: true,
      hasGroup: false,
    }
  }).then(res => {
    if (res.code === '200') {
      data.flashData = res.data
      data.flashData.forEach(item => {
        item.flashDown = 9
      })
      setInterval(flashDown, 100)
    } else {
      ElMessage.error(res.msg)
    }
  })
}
const loadGroup = (pageNum) => {
  request.get('/orders/selectGroupPage', {
    params: {
      pageNum: pageNum,
      pageSize: 4,
      type: 'GROUP',
    }
  }).then(res => {
    if (res.code === '200') {
      data.groupData = res.data?.list
      data.pages = res.data.pages
      data.groupData.forEach(item => {
        item.groupDown = 9
      })
      setInterval(groupDown, 100)
    }
  })
}
const getGroup = (index) => {
  loadGroup(index + 1)
}
const navTo = (url) => {
  location.href = url
}

const flashDown = () => {
  data.flashData?.forEach(item => {
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

const groupDown = () => {
  data.groupData?.forEach(item => {
    let maxTime = item.maxTime
    if (maxTime > 0) {
      let remain = Math.floor(maxTime % 3600)
      item.hour = Math.floor(maxTime / 3600)
      item.minutes = Math.floor(remain / 60)
      item.seconds = Math.floor(remain % 60)
      if (item.groupDown === 0) {
        item.maxTime--
        item.groupDown = 9
      } else {
        item.groupDown--
      }

    }
  })
}

const handleGroupChange = (index) => {
  currentPage.value = index + 1
  getGroup(index + 1)
}

loadCarouse()
loadGoods()
loadFlash()
loadGroup(1)

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

/* 轮播图容器样式 */
.carousel-wrapper {
  width: 100%;
  border-radius: 10px;
  overflow: hidden;
}

/* 轮播项样式 */
.carousel-item {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f5f5f5; /* 设置背景色，当图片未完全填充时可见 */
}

/* 图片样式 */
.carousel-image {
  width: 100%;
  height: 100%;
  object-fit: contain; /* 保持图片原始比例，确保完整显示 */
  object-position: center; /* 居中显示图片 */
  background-color: #f5f5f5; /* 可选：添加背景色 */
}

/* 如果您想让图片填充整个容器但保持比例，可以使用 cover */
/*
.carousel-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center;
}
*/

/* 优化 Element Plus 轮播图样式 */
:deep(.el-carousel__container) {
  height: 100%;
}

:deep(.el-carousel__item) {
  height: 100%;
}

.carousel-caption {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 20px;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.7));
  color: white;
}

.carousel-caption h3 {
  margin: 0;
  font-size: 24px;
  font-weight: bold;
}

.carousel-caption p {
  margin: 10px 0 0;
  font-size: 16px;
  opacity: 0.9;
}

/* 自定义 Element Plus 轮播图指示器样式 */
:deep(.el-carousel__indicators) {
  transform: translateY(16px);
}

:deep(.el-carousel__indicator) {
  padding: 12px 4px;
}

:deep(.el-carousel__button) {
  width: 30px;
  height: 3px;
  border-radius: 3px;
  background-color: rgba(255, 255, 255, 0.7);
  transition: all 0.3s;
}

:deep(.el-carousel__indicator.is-active .el-carousel__button) {
  background-color: #fff;
  width: 40px;
}

/* 优化轮播图箭头样式 */
:deep(.el-carousel__arrow) {
  background-color: rgba(0, 0, 0, 0.3);
  border-radius: 50%;
  width: 36px;
  height: 36px;
  transform: translateX(0);
  transition: all 0.3s;
}

:deep(.el-carousel__arrow:hover) {
  background-color: rgba(0, 0, 0, 0.5);
}

:deep(.el-carousel__arrow--left) {
  transform: translateX(20px);
}

:deep(.el-carousel__arrow--right) {
  transform: translateX(-20px);
}

.carousel-wrapper:hover {
  :deep(.el-carousel__arrow--left) {
    transform: translateX(0);
  }
  
  :deep(.el-carousel__arrow--right) {
    transform: translateX(0);
  }
}

.group-carousel-container {
  background-color: #faf7f0;
  height: 350px;
  border-radius: 12px;
  padding: 15px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.group-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.group-title {
  font-size: 18px;
  font-weight: bold;
  color: #333;
}

.group-indicators {
  display: flex;
  gap: 6px;
}

.indicator-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background-color: #ddd;
  transition: all 0.3s;
}

.indicator-dot.active {
  width: 18px;
  border-radius: 3px;
  background-color: #faa303;
}

.group-carousel {
  border-radius: 8px;
  overflow: hidden;
}

.group-item {
  display: flex;
  align-items: center;
  padding: 12px;
  margin: 8px 0;
  background-color: #fff;
  border-radius: 8px;
  transition: all 0.3s;
}

.group-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

.group-item-image {
  width: 70px;
  height: 70px;
  border-radius: 8px;
  object-fit: cover;
  border: 1px solid #eee;
}

.group-item-info {
  flex: 1;
  margin-left: 15px;
}

.group-item-title {
  font-weight: bold;
  font-size: 16px;
  margin-bottom: 8px;
  color: #333;
}

.group-item-price {
  color: #ff4d4f;
  font-weight: bold;
  font-size: 15px;
}

.group-item-user {
  display: flex;
  align-items: center;
  margin-left: 20px;
  min-width: 180px;
}

.user-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 2px solid #fff;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.user-name {
  margin: 0 10px;
  color: #666;
  font-size: 14px;
}

.group-status {
  color: #faa303;
  font-size: 13px;
}

.group-item-countdown {
  color: #ff4d4f;
  margin: 0 15px;
  font-size: 14px;
}

.join-group-btn {
  background-color: #faa303;
  border-color: #faa303;
  color: white;
  border-radius: 20px;
  padding: 8px 16px;
  transition: all 0.3s;
}

.join-group-btn:hover {
  background-color: #e89503;
  border-color: #e89503;
  transform: translateY(-2px);
}

/* 自定义 Element Plus 轮播图过渡效果 */
:deep(.el-carousel__item) {
  overflow: auto;
  scrollbar-width: none;  /* Firefox */
}

:deep(.el-carousel__item::-webkit-scrollbar) {
  display: none;  /* Chrome Safari */
}
</style>