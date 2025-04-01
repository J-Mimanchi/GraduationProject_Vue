<template>
  <div class="manager-container">
    <!-- 侧边栏 -->
    <aside class="manager-sidebar">
      <div class="sidebar-header">
        <img src="@/assets/imgs/logo.jpg" alt="MyShop Logo">
        <div class="title">MyShop</div>
      </div>
      <nav class="sidebar-nav">
        <el-menu
          :default-active="router.currentRoute.value.path"
          :default-openeds="['/manager/home', 'info']"
          router
          class="sidebar-menu"
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
      </nav>
    </aside>

    <!-- 主要内容区 -->
    <div class="manager-main">
      <!-- 顶部导航栏 -->
      <header class="manager-header">
        <div class="header-left">
          <el-icon class="toggle-sidebar" @click="isCollapse = !isCollapse"><Fold /></el-icon>
          <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/manager/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>{{ currentPath }}</el-breadcrumb-item>
          </el-breadcrumb>
        </div>
        <div class="header-right">
          <el-menu mode="horizontal" :default-active="router.currentRoute.value.path" router>
            <el-menu-item index="/front/home">
              <el-icon><House /></el-icon>
              返回前台
            </el-menu-item>
          </el-menu>
          <el-dropdown>
            <div class="user-info">
              <img :src="data.user?.avatar || '@/assets/imgs/avatar.png'" alt="用户头像">
              <span>{{ data.user?.name }}</span>
              <el-icon><arrow-down /></el-icon>
            </div>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item @click="router.push('/manager/person')">
                  <el-icon><User /></el-icon>
                  <span>个人资料</span>
                </el-dropdown-item>
                <el-dropdown-item @click="router.push('/manager/password')">
                  <el-icon><Lock /></el-icon>
                  <span>修改密码</span>
                </el-dropdown-item>
                <el-dropdown-item divided @click="logout">
                  <el-icon><SwitchButton /></el-icon>
                  <span>退出登录</span>
                </el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
        </div>
      </header>

      <!-- 内容区域 -->
      <main class="manager-content">
        <RouterView @updateUser="updateUser" />
      </main>
    </div>
  </div>
</template>

<script setup>
import router from "@/router/index.js";
import {reactive, ref, computed} from "vue";
import { 
  HomeFilled, 
  UserFilled, 
  User, 
  Lock, 
  SwitchButton,
  ArrowDown,
  Fold,
  House
} from '@element-plus/icons-vue'

const data = reactive({
  user: JSON.parse(localStorage.getItem('myshop-user') || '{}')
})

const logout = () => {
  localStorage.removeItem('myshop-user')
  location.href = '/login'
}

const updateUser = () => {
  data.user = JSON.parse(localStorage.getItem('myshop-user') || '{}')
}

const isCollapse = ref(false);
const currentPath = computed(() => router.currentRoute.value.meta.name);
</script>

<style scoped>
.manager-container {
  display: flex;
  min-height: 100vh;
  background-color: #f8fafc;
}

/* 侧边栏样式 */
.manager-sidebar {
  width: 240px;
  background-color: #f1f5f9;
  color: #64748b;
  display: flex;
  flex-direction: column;
  position: fixed;
  height: 100vh;
  left: 0;
  top: 0;
  z-index: 1000;
  transition: all 0.3s ease;
  border-right: 1px solid #e2e8f0;
}

.sidebar-header {
  height: 60px;
  display: flex;
  align-items: center;
  padding: 0 20px;
  background-color: #fff;
  border-bottom: 1px solid #e2e8f0;
}

.sidebar-header img {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  margin-right: 12px;
}

.sidebar-header .title {
  font-size: 18px;
  font-weight: 600;
  color: #475569;
}

.sidebar-nav {
  flex: 1;
  overflow-y: auto;
  padding: 20px 0;
}

.sidebar-menu {
  border-right: none;
  background-color: transparent;
}

.sidebar-menu :deep(.el-menu-item),
.sidebar-menu :deep(.el-sub-menu__title) {
  color: #64748b;
}

.sidebar-menu :deep(.el-menu-item:hover),
.sidebar-menu :deep(.el-sub-menu__title:hover) {
  background-color: #e2e8f0;
  color: #475569;
}

.sidebar-menu :deep(.el-menu-item.is-active) {
  background-color: #e2e8f0;
  color: #475569;
  font-weight: 500;
}

.sidebar-menu :deep(.el-sub-menu .el-menu-item) {
  color: #64748b;
}

.sidebar-menu :deep(.el-sub-menu .el-menu-item:hover) {
  background-color: #e2e8f0;
  color: #475569;
}

.sidebar-menu :deep(.el-sub-menu .el-menu-item.is-active) {
  background-color: #e2e8f0;
  color: #475569;
  font-weight: 500;
}

/* 主要内容区样式 */
.manager-main {
  flex: 1;
  margin-left: 240px;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background-color: #f8fafc;
}

/* 顶部导航栏样式 */
.manager-header {
  height: 60px;
  background-color: #fff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  position: fixed;
  top: 0;
  right: 0;
  left: 240px;
  z-index: 999;
  border-bottom: 1px solid #e2e8f0;
}

.header-left {
  display: flex;
  align-items: center;
}

.header-left :deep(.el-breadcrumb__inner) {
  color: #64748b;
}

.header-left :deep(.el-breadcrumb__item:last-child .el-breadcrumb__inner) {
  color: #475569;
  font-weight: 500;
}

.header-right {
  display: flex;
  align-items: center;
}

.header-right :deep(.el-menu) {
  background: transparent;
  border: none;
  margin-right: 16px;
}

.header-right :deep(.el-menu-item) {
  height: 60px;
  line-height: 60px;
  padding: 0 16px;
  color: #64748b;
}

.header-right :deep(.el-menu-item:hover) {
  color: #475569;
  background: #f1f5f9;
}

.header-right :deep(.el-menu-item .el-icon) {
  margin-right: 4px;
  font-size: 16px;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  padding: 0 12px;
  height: 40px;
  border-radius: 6px;
  transition: all 0.3s ease;
  color: #64748b;
}

.user-info:hover {
  background-color: #f1f5f9;
  color: #475569;
}

.user-info img {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  object-fit: cover;
  border: 2px solid #e2e8f0;
}

.user-info span {
  font-size: 14px;
}

.user-info .el-icon {
  font-size: 12px;
  margin-left: 4px;
}

/* 内容区域样式 */
.manager-content {
  flex: 1;
  padding: 80px 20px 20px;
  background-color: #f8fafc;
  min-height: calc(100vh - 60px);
}

/* 下拉菜单样式 */
:deep(.el-dropdown-menu) {
  padding: 4px 0;
  border-radius: 6px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 0 2px 4px -1px rgba(0, 0, 0, 0.03);
}

:deep(.el-dropdown-menu__item) {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  color: #64748b;
  font-size: 14px;
}

:deep(.el-dropdown-menu__item .el-icon) {
  font-size: 16px;
}

:deep(.el-dropdown-menu__item:hover) {
  background-color: #f1f5f9;
  color: #475569;
}

:deep(.el-dropdown-menu__item.is-divided) {
  margin-top: 4px;
  border-top: 1px solid #e2e8f0;
}

/* 响应式布局 */
@media screen and (max-width: 768px) {
  .manager-sidebar {
    transform: translateX(-100%);
  }

  .manager-main {
    margin-left: 0;
  }

  .manager-header {
    left: 0;
  }
}

/* 滚动条样式 */
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}
</style>