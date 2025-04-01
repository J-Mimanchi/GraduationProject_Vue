<template>
  <div class="front-container">
    <!--头部-->
    <header class="front-header">
      <div class="front-header-left" @click="$router.push('/front/home')">
        <img src="@/assets/imgs/logo.jpg" alt="MyShop Logo">
        <div class="title">MyShop</div>
      </div>
      <div class="front-header-center">
        <nav class="front-header-nav">
          <el-menu :default-active="router.currentRoute.value.path" mode="horizontal" router>
            <el-menu-item index="/front/home">首页</el-menu-item>
            <el-menu-item index="/front/group">团购专区</el-menu-item>
            <el-menu-item index="/front/flash">秒杀专区</el-menu-item>
            <el-menu-item index="/front/person">个人中心</el-menu-item>
            <el-menu-item v-if="data.user.role === 'ADMIN'" index="/manager/home">
              <el-icon><Setting /></el-icon>
              管理后台
            </el-menu-item>
          </el-menu>
        </nav>
        <div class="search-container" v-if="router.currentRoute.value.path === '/front/home'">
          <el-input
            v-model="data.goodsName"
            class="search-input"
            placeholder="搜索商品..."
            @keyup.enter="search"
          >
            <template #prefix>
              <el-icon><Search /></el-icon>
            </template>
          </el-input>
          <el-button type="primary" class="search-button" @click="search">
            <el-icon><Search /></el-icon>
            <span>搜索</span>
          </el-button>
        </div>
      </div>
      <div class="front-header-right">
        <template v-if="!data.user.username">
          <el-button class="login-btn" @click="router.push('/login')">登录</el-button>
          <el-button class="register-btn" @click="router.push('/register')">注册</el-button>
        </template>
        <template v-else>
          <el-dropdown>
            <div class="front-header-dropdown">
              <img :src="data.user.avatar" alt="用户头像">
              <div class="user-info">
                <span>{{ data.user.name }}</span>
                <el-icon><arrow-down /></el-icon>
              </div>
            </div>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item @click="logout">退出登录</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
        </template>
      </div>
    </header>

    <!--主体-->
    <main class="main-body">
      <router-view ref="child" @updateUser="updateUser" />
    </main>

    <!--底部-->
    <footer class="front-footer">
      <div class="footer-bottom">
        <p>© XingBaoJiang</p>
      </div>
    </footer>
  </div>
</template>

<script setup>
import {reactive} from "vue";
import router from "@/router";
import {ElMessage} from "element-plus";
import { Search, ChatDotRound, Share, Star, Setting } from '@element-plus/icons-vue'

const data = reactive({
  user: JSON.parse(localStorage.getItem("myshop-user") || '{}'),
  goodsName: "",
})

// 添加调试信息
console.log('当前用户信息：', data.user)
console.log('用户角色：', data.user.role)

const updateUser = () => {
  data.user = JSON.parse(localStorage.getItem('myshop-user') || '{}')
}

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
.front-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.front-header {
  display: flex;
  align-items: center;
  height: 60px;
  background: linear-gradient(135deg, #f5f7fa 0%, #ffffff 100%);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  padding: 0 20px;
}

.front-header-left {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.front-header-left img {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  object-fit: cover;
}

.front-header-left .title {
  color: #333;
  font-size: 18px;
  font-weight: 600;
}

.front-header-center {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
}

.front-header-nav {
  flex: 1;
  display: flex;
  justify-content: center;
}

.front-header-nav :deep(.el-menu) {
  background: transparent;
  border: none;
  display: flex;
  justify-content: center;
  flex-wrap: nowrap;
  width: auto;
}

.front-header-nav :deep(.el-menu--horizontal) {
  border-bottom: none;
  display: flex;
  flex-wrap: nowrap;
}

.front-header-nav :deep(.el-menu--horizontal > .el-menu-item) {
  border-bottom: none;
  color: #666;
  font-size: 15px;
  height: 60px;
  line-height: 60px;
  padding: 0 20px;
  transition: all 0.3s ease;
  white-space: nowrap;
}

.front-header-nav :deep(.el-menu--horizontal > .el-menu-item:hover) {
  color: #8B5CF6;
  background: rgba(139, 92, 246, 0.05);
}

.front-header-nav :deep(.el-menu--horizontal > .el-menu-item.is-active) {
  color: #8B5CF6;
  background: rgba(139, 92, 246, 0.1);
  font-weight: 500;
  border-bottom: 2px solid #8B5CF6;
}

.front-header-nav :deep(.el-menu--horizontal > .el-menu-item:not(.is-disabled):focus),
.front-header-nav :deep(.el-menu--horizontal > .el-menu-item:not(.is-disabled):active) {
  background-color: transparent;
}

.search-container {
  display: flex;
  align-items: center;
  background-color: rgba(139, 92, 246, 0.05);
  border-radius: 6px;
  overflow: hidden;
  margin-left: 20px;
  padding: 4px;
  gap: 8px;
}

.search-input {
  width: 200px;
}

.search-input :deep(.el-input__wrapper) {
  background-color: #fff;
  border: none;
  box-shadow: none !important;
  border-radius: 4px;
}

.search-input :deep(.el-input__inner) {
  color: #333;
}

.search-input :deep(.el-input__prefix) {
  color: #8B5CF6;
}

.search-button {
  display: flex;
  align-items: center;
  gap: 4px;
  background-color: #f5f7fa;
  color: #666;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  transition: all 0.3s ease;
  box-shadow: none !important;
  height: 100%;
}

.search-button:hover {
  background-color: #e4e7ed;
  color: #333;
  transform: translateY(-1px);
}

.front-header-right {
  display: flex;
  align-items: center;
  gap: 8px;
}

.front-header-dropdown {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.front-header-dropdown img {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  object-fit: cover;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 4px;
  color: #333;
}

.login-btn, .register-btn {
  padding: 8px 16px;
  border: 1px solid #dcdfe6;
  background: #fff;
  color: #666;
  font-weight: 500;
  transition: all 0.3s ease;
  border-radius: 4px;
}

.login-btn:hover, .register-btn:hover {
  background: #f5f7fa;
  border-color: #c0c4cc;
  color: #333;
  transform: translateY(-1px);
}

.register-btn {
  background: #f5f7fa;
  color: #666;
  border-color: #dcdfe6;
}

.register-btn:hover {
  background: #e4e7ed;
  color: #333;
  border-color: #c0c4cc;
}

.main-body {
  flex: 1;
  padding: 70px 20px 20px;
  min-height: calc(100vh - 60px - 200px);
  background-color: var(--background-color);
}

.front-footer {
  background-color: #2c3e50;
  color: #fff;
  padding: 20px;
  text-align: center;
}

.footer-bottom {
  color: rgba(255, 255, 255, 0.7);
}

@media screen and (max-width: 768px) {
  .front-header-center {
    display: none;
  }
  
  .search-container {
    display: none;
  }
  
  .footer-content {
    grid-template-columns: 1fr;
    text-align: center;
  }
  
  .social-links {
    justify-content: center;
  }
}

.el-dropdown-menu :deep(.el-dropdown-menu__item) {
  display: flex;
  align-items: center;
  gap: 8px;
}

.el-dropdown-menu :deep(.el-icon) {
  font-size: 16px;
}

.front-header-nav :deep(.el-menu-item .el-icon) {
  margin-right: 4px;
  font-size: 16px;
}
</style>