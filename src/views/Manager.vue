<template>
  <div class="manager-container">
    <!--  头部开始  -->
    <div class="manager-header">
      <div class="manager-header-left">
        <!-- 标题和Logo -->
        <img src="@/assets/imgs/logo.jpg" alt="">
        <div class="title">MyShop</div>
      </div>
      <div class="manager-header-center">
        <!-- 面包屑 -->
        <el-breadcrumb separator="/">
          <el-breadcrumb-item to="/manager/home">首页</el-breadcrumb-item>
          <el-breadcrumb-item>{{ router.currentRoute.value.meta.name }}</el-breadcrumb-item>
        </el-breadcrumb>
      </div>
      <div class="manager-header-right">
        <!-- 右侧头像和用户名 -->
        <el-dropdown style="cursor: pointer;">
          <div style="padding-right: 20px; display: flex; align-items: center;">
            <img v-if="data.user.avatar" :src="data.user?.avatar" alt="" style="width: 40px; height: 40px; display: block; border-radius: 50%">
            <img v-else src="@/assets/imgs/avatar.png" alt="" style="width: 40px; height: 40px; display: block; border-radius: 50%">
            <span style="margin-left: 5px; color: white">{{ data.user?.name }}</span>
          </div>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item @click.native="router.push('/manager/person')">个人资料</el-dropdown-item>
              <el-dropdown-item @click.native="router.push('/manager/password')">修改密码</el-dropdown-item>
              <el-dropdown-item @click.native="logout">退出登录</el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </div>
    </div>
    <!--  头部结束 -->

    <!-- 下面区域开始 -->
    <div style="display: flex">
      <div class="manager-main-left">
        <!-- 导航菜单区域 -->
        <el-menu
            :default-active="router.currentRoute.value.path"
            :default-openeds="['/manager/home', 'info']"
            router
        >
          <el-menu-item index="/manager/home">
            <el-icon><home-filled /></el-icon>
            <span>系统首页</span>
          </el-menu-item>
          <el-sub-menu index="info">
            <template #title>
              <el-icon><UserFilled /></el-icon>
              <span>信息管理</span>
            </template>
            <el-menu-item index="/manager/orders">订单信息</el-menu-item>
            <el-menu-item index="/manager/carousel">广告信息</el-menu-item>
            <el-menu-item index="/manager/category">商品分类</el-menu-item>
            <el-menu-item index="/manager/goods">商品信息</el-menu-item>
            <el-menu-item index="/manager/comment">评价信息</el-menu-item>
            <el-menu-item index="/manager/logs">系统日志</el-menu-item>
            <el-menu-item index="/manager/user">用户信息</el-menu-item>
          </el-sub-menu>
        </el-menu>
      </div>

      <div class="manager-main-right">
        <!-- 主体内容区域 -->
        <RouterView @updateUser="updateUser" />
      </div>
    </div>
    <!-- 下面区域结束 -->

  </div>
</template>

<script setup>
import router from "@/router/index.js";
import {reactive} from "vue";

const data = reactive({
  user: JSON.parse(localStorage.getItem('myshop-user') || '{}')
})

const logout = () => {
  localStorage.removeItem('myshop-user')
  location.href = '/login'
}

const updateUser = () => {
  // 获取最新的缓存数据
  data.user = JSON.parse(localStorage.getItem('myshop-user') || '{}')
}
</script>

<style scoped>
@import "@/assets/css/manager.css";
</style>