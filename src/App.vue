<template>
  <div class="app-container">
    <Header 
      v-if="!isLoginPage && !isSignupPage && !isFindPasswordPage" 
      @search="handleHeaderSearch"
      @reset="handleHeaderReset"
    />
    <div class="main-layout" v-if="!isLoginPage && !isSignupPage && !isFindPasswordPage">
      <Sidebar 
        v-model:selectedCategory="selectedCategory"
        :isMypage="isMypagePage"
      />
      <router-view v-slot="{ Component }">
        <component :is="Component" ref="currentComponent" />
      </router-view>
    </div>
    <router-view v-else v-slot="{ Component }">
      <component :is="Component" ref="currentComponent" />
    </router-view>
  </div>
</template>

<script setup>
import Header from './components/Header.vue'
import Sidebar from './components/Sidebar.vue'
import { computed, ref } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const currentComponent = ref(null)
const selectedCategory = ref(null)

const isLoginPage = computed(() => route.path === '/login')
const isSignupPage = computed(() => route.path === '/signup')
const isFindPasswordPage = computed(() => route.path === '/find-password')
const isMypagePage = computed(() => route.path === '/mypage')
const handleHeaderSearch = (query) => {
  // 현재 라우트가 메인 페이지인 경우에만 검색 실행
  if (route.path === '/') {
    if (currentComponent.value?.handleSearch) {
      currentComponent.value.handleSearch(query)
    }
  }
}

const handleHeaderReset = () => {
  // 현재 라우트가 메인 페이지인 경우에만 리셋 실행
  if (route.path === '/') {
    if (currentComponent.value?.handleReset) {
      currentComponent.value.handleReset()
    }
  }
}
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
  overflow: hidden;
}

body {
  background-color: #faf7f5;
}

.app-container {
  height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: #faf7f5;
  overflow: hidden;
}

/* router-view가 있는 영역이 남은 공간을 모두 차지하도록 설정 */
.app-container > :nth-child(2) {
  flex: 1;
  overflow: hidden;
}

/* 스크롤이 필요한 컨테이너에 대한 스타일 */
.scrollable {
  overflow-y: auto;
  overflow-x: hidden;
}

.main-layout {
  display: flex;
  flex: 1;
}

.main-layout > :last-child {
  flex: 1;
}
</style>
