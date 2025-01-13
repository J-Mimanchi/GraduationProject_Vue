<template>
  <div>
    <!--头部-->
    <div class="front-header">
      <div class="front-header-left" @click="$router.push('/front/home')">
        <img src="@/assets/imgs/logo.jpg" alt="">
        <div class="title">MyShop</div>
      </div>
      <div class="front-header-center">
        <div class="front-header-nav" style="flex: 1">
          <el-menu :default-active="router.currentRoute.value.path" mode="horizontal" router>
						<el-menu-item index="/front/home">首页</el-menu-item>
						<el-menu-item index="/front/group">团购专区</el-menu-item>
						<el-menu-item index="/front/flash">秒杀专区</el-menu-item>
						<el-menu-item index="/front/orders">订单中心</el-menu-item>
						<el-menu-item index="/front/comment">评价中心</el-menu-item>
						<el-menu-item index="/front/person">个人中心</el-menu-item>
          </el-menu>
        </div>
        <div style="width: 300px; background-color: #9f8be3; height: 60px" v-if="router.currentRoute.value.path === '/front/home'">
          <el-input v-model="data.goodsName" style="width: 200px; margin-right: 5px" placeholder="请输入商品名称"></el-input>
          <el-button type="warning" @click="search">搜索</el-button>
        </div>
      </div>
      <div class="front-header-right">
        <div v-if="!data.user.username">
          <el-button @click="router.push('/login')">登录</el-button>
          <el-button @click="router.push('/register')">注册</el-button>
        </div>
        <div v-else>
          <el-dropdown>
            <div class="front-header-dropdown">
              <img :src="data.user.avatar" alt="">
              <div style="margin-left: 10px; color: white">
                <span>{{ data.user.name }}</span><el-icon><arrow-down /></el-icon>
              </div>
            </div>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item @click.native="logout">退出</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
        </div>
      </div>
    </div>
    <!--主体-->
    <div class="main-body" style="min-height: 550px">
      <router-view ref="child" @updateUser="updateUser" />
    </div>


    <!--底部-->
    <div class="front-footer" style="padding: 10px 0">
      <div style="text-align: center; color: #666666">© XingBaoJiang</div>
    </div>
  </div>

</template>

<script setup>
import {reactive} from "vue";
import router from "@/router";
import {ElMessage} from "element-plus";

const data = reactive({
  user: JSON.parse(localStorage.getItem("myshop-user") || '{}'),
  goodsName: "",
})

const updateUser = () => {
  data.user = JSON.parse(localStorage.getItem('myshop-user') || '{}')   // 重新获取下用户的最新信息
}

// 退出登录
const logout = () => {
  localStorage.removeItem("myshop-user");
  router.push("/login");
}

const search = () => {
  if (data.goodsName) {
    location.href = '/front/goods?name=' + data.goodsName
  } else {
    ElMessage.info('请输入搜索内容')
  }
}
</script>

<style scoped>
  @import "@/assets/css/front.css";
</style>