<template>
  <div class="page-wrapper">
    <div class="mypage-container">
      <!-- 메인 콘텐츠 영역 -->
      <main class="main-content">
        <!-- 내 정보 수정 -->
        <div v-if="activeMenu === 'profile'" class="content-section">
          <div class="content-header">
            <h2 class="section-title">내 정보 수정</h2>
          </div>
          <div class="profile-form-container">
            <form @submit.prevent="updateProfile" class="profile-form">
              <div class="form-group">
                <label for="nickname">닉네임</label>
                <input type="text" id="nickname" v-model="profile.nickname" class="form-control">
              </div>
              <div class="form-group">
                <label for="email">이메일</label>
                <input type="email" id="email" v-model="profile.email" class="form-control">
              </div>
              <div class="form-group">
                <label for="phone">전화번호</label>
                <input type="tel" id="phone" v-model="profile.phone" class="form-control">
              </div>
              <button type="submit" class="btn btn-primary">정보 수정</button>
            </form>
          </div>
        </div>

        <!-- 내가 만든 스터디 -->
        <div v-if="activeMenu === 'created'" class="content-section">
          <div class="content-header">
            <h2 class="section-title">내가 만든 스터디</h2>
            <div class="study-tabs">
              <button 
                class="tab-btn" 
                :class="{ active: studyTab === 'all' }"
                @click="studyTab = 'all'"
              >
                전체
              </button>
              <button 
                class="tab-btn" 
                :class="{ active: studyTab === 'recruiting' }"
                @click="studyTab = 'recruiting'"
              >
                모집중
              </button>
              <button 
                class="tab-btn" 
                :class="{ active: studyTab === 'completed' }"
                @click="studyTab = 'completed'"
              >
                모집완료
              </button>
            </div>
          </div>
          <div class="study-list">
            <div v-for="study in filteredCreatedStudies" :key="study.id" class="study-card" @click="goToStudyDetail(study.id)">
              <div class="study-thumbnail">
                <div v-show="study.isImageLoading" class="study-thumbnail-skeleton">
                  <div class="skeleton-content"></div>
                </div>
                <img 
                  v-show="!study.isImageLoading"
                  :src="study.thumbnail || logoImage" 
                  :alt="study.title" 
                  loading="lazy" 
                  decoding="async" 
                  fetchpriority="high"
                  width="400"
                  height="300"
                  sizes="(max-width: 768px) 100vw, 25vw"
                  @load="handleImageLoad(study)"
                  @error="handleImageError(study)"
                >
              </div>
              <div class="study-info">
                <h3 class="study-title">{{ study.title }}</h3>
                <p class="study-content">{{ study.content }}</p>
                <div class="study-meta">
                  <span class="study-members">{{ study.currentMembers }}/{{ study.maxMembers }}명</span>
                  <span class="study-status" :class="study.status">{{ study.status }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- 신청한 스터디 -->
        <div v-if="activeMenu === 'applied'" class="content-section">
          <div class="content-header">
            <h2 class="section-title">신청한 스터디</h2>
            <div class="study-tabs">
              <button 
                class="tab-btn" 
                :class="{ active: appliedTab === 'all' }"
                @click="appliedTab = 'all'"
              >
                전체
              </button>
              <button 
                class="tab-btn" 
                :class="{ active: appliedTab === 'waiting' }"
                @click="appliedTab = 'waiting'"
              >
                승인대기
              </button>
              <button 
                class="tab-btn" 
                :class="{ active: appliedTab === 'approved' }"
                @click="appliedTab = 'approved'"
              >
                승인
              </button>
              <button 
                class="tab-btn" 
                :class="{ active: appliedTab === 'rejected' }"
                @click="appliedTab = 'rejected'"
              >
                거절
              </button>
            </div>
          </div>
          <div class="study-list">
            <div v-for="study in filteredAppliedStudies" :key="study.id" class="study-card" @click="goToStudyDetail(study.id)">
              <div class="study-thumbnail">
                <div v-show="study.isImageLoading" class="study-thumbnail-skeleton">
                  <div class="skeleton-content"></div>
                </div>
                <img 
                  v-show="!study.isImageLoading"
                  :src="study.thumbnail || logoImage" 
                  :alt="study.title" 
                  loading="lazy" 
                  decoding="async" 
                  fetchpriority="high"
                  width="400"
                  height="300"
                  sizes="(max-width: 768px) 100vw, 25vw"
                  @load="handleImageLoad(study)"
                  @error="handleImageError(study)"
                >
              </div>
              <div class="study-info">
                <h3 class="study-title">{{ study.title }}</h3>
                <p class="study-content">{{ study.content }}</p>
                <div class="study-meta">
                  <span class="study-members">{{ study.currentMembers }}/{{ study.maxMembers }}명</span>
                  <span class="study-status" :class="study.applicationStatus">{{ study.applicationStatus }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </main>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import logoImage from '@/assets/logo.png'

const router = useRouter()
const route = useRoute()
const activeMenu = ref('applied')
const username = ref('홍길동') // TODO: 실제 사용자 이름으로 대체
const isLoggedIn = ref(true)

// URL의 tab 파라미터에 따라 메뉴 활성화
watch(() => route.query.tab, (newTab) => {
  if (newTab === 'profile') {
    activeMenu.value = 'profile'
  } else if (newTab === 'created') {
    activeMenu.value = 'created'
  } else if (newTab === 'applied') {
    activeMenu.value = 'applied'
  }
}, { immediate: true })

// 프로필 정보
const profile = ref({
  nickname: '',
  email: '',
  phone: ''
})

// 내가 만든 스터디 목록
const createdStudies = ref([])

// 신청한 스터디 목록
const appliedStudies = ref([])

const studyTab = ref('all')
const appliedTab = ref('all')

// 내가 만든 스터디 필터링
const filteredCreatedStudies = computed(() => {
  if (studyTab.value === 'all') return createdStudies.value
  return createdStudies.value.filter(study => {
    if (studyTab.value === 'recruiting') return study.status === '모집중'
    if (studyTab.value === 'completed') return study.status === '모집완료'
    return true
  })
})

// 신청한 스터디 필터링
const filteredAppliedStudies = computed(() => {
  if (appliedTab.value === 'all') return appliedStudies.value
  return appliedStudies.value.filter(study => {
    if (appliedTab.value === 'waiting') return study.applicationStatus === '승인대기'
    if (appliedTab.value === 'approved') return study.applicationStatus === '승인'
    if (appliedTab.value === 'rejected') return study.applicationStatus === '거절'
    return true
  })
})

// 프로필 정보 수정
const updateProfile = () => {
  // TODO: API 호출로 대체
  alert('프로필 정보가 수정되었습니다.')
}

// 스터디 상세 페이지로 이동
const goToStudyDetail = (studyId) => {
  const study = [...createdStudies.value, ...appliedStudies.value].find(s => s.id === studyId)
  if (!study) return

  if (activeMenu.value === 'applied') {
    router.push({
      path: `/study/${studyId}`,
      query: { tab: 'applied' }
    })
  } else if (activeMenu.value === 'created') {
    router.push({
      path: `/study/${studyId}`,
      query: { tab: 'created' }
    })
  } else {
    router.push({
      path: `/study/${studyId}`
    })
  }
}

// 뒤로가기 함수
const goBack = () => {
  router.back()
}

// 로그아웃 함수
const logout = () => {
  // TODO: 로그아웃 로직 구현
  alert('로그아웃되었습니다.')
}

// 초기 데이터 로드
onMounted(() => {
  // TODO: API 호출로 대체
  // 임시 데이터
  profile.value = {
    nickname: '홍길동',
    email: 'hong@example.com',
    phone: '010-1234-5678'
  }

  createdStudies.value = [
    {
      id: 1,
      title: '프로그래밍 스터디',
      content: '함께 프로그래밍을 배우고 실력을 향상시켜요!',
      thumbnail: 'https://picsum.photos/400/300',
      currentMembers: 3,
      maxMembers: 5,
      status: '모집중',
      isImageLoading: true
    },
    {
      id: 2,
      title: '디자인 스터디',
      content: 'UI/UX 디자인을 함께 배워요',
      thumbnail: 'https://picsum.photos/400/301',
      currentMembers: 5,
      maxMembers: 5,
      status: '모집완료',
      isImageLoading: true
    }
  ]

  appliedStudies.value = [
    {
      id: 3,
      title: '알고리즘 스터디',
      content: '코딩 테스트 대비 알고리즘 문제 풀이',
      thumbnail: 'https://picsum.photos/400/302',
      currentMembers: 4,
      maxMembers: 6,
      applicationStatus: '승인대기',
      isImageLoading: true
    },
    {
      id: 4,
      title: '영어 회화 스터디',
      content: '매일 영어로 대화하며 실력 향상',
      thumbnail: 'https://picsum.photos/400/303',
      currentMembers: 3,
      maxMembers: 4,
      applicationStatus: '승인',
      isImageLoading: true
    },
    {
      id: 5,
      title: '마케팅 스터디',
      content: '디지털 마케팅 전략과 실전 사례',
      thumbnail: 'https://picsum.photos/400/304',
      currentMembers: 6,
      maxMembers: 6,
      applicationStatus: '거절',
      isImageLoading: true
    }
  ]

  // Add preload link for logo image
  const link = document.createElement('link')
  link.rel = 'preload'
  link.as = 'image'
  link.href = logoImage
  document.head.appendChild(link)

  // 이미지 사전 로드
  const preloadImages = (studies) => {
    studies.forEach(study => {
      if (study.thumbnail) {
        const img = new Image()
        img.onload = () => {
          study.isImageLoading = false
        }
        img.onerror = () => {
          study.isImageLoading = false
          study.thumbnail = logoImage
        }
        img.src = study.thumbnail
      }
    })
  }

  preloadImages(createdStudies.value)
  preloadImages(appliedStudies.value)
})

// 이미지 로드 핸들러
const handleImageLoad = (study) => {
  study.isImageLoading = false
}

// 이미지 에러 핸들러
const handleImageError = (study) => {
  study.isImageLoading = false
  study.thumbnail = logoImage
}
</script>

<style scoped>
.page-wrapper {
  min-height: 100vh;
  background-color: #faf7f5;
}

.mypage-container {
  display: flex;
  min-height: calc(100vh - 60px);
  background-color: #faf7f5;
  margin-top: 0; /* 헤더와의 간격 제거 */
}

.sidebar {
  width: 250px;
  background-color: #fbf9f8;
  padding: 2rem 1rem;
  border-right: 1px solid #eee5dd;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  min-height: calc(100vh - 60px);
}

.sidebar-title {
  color: #6f4e37;
  font-weight: 600;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #eee5dd;
  text-align: center;
}

.category-list {
  list-style: none;
  padding: 0;
  margin: 0;
  text-align: center;
}

.category-item {
  margin-bottom: 0.5rem;
}

.category-item a {
  display: block;
  padding: 0.5rem;
  color: #4b3621;
  text-decoration: none;
  border-radius: 6px;
  transition: all 0.2s ease;
}

.category-item a:hover {
  background-color: #eee5dd;
  color: #6f4e37;
}

.category-item.selected a {
  background-color: #eee5dd;
  color: #6f4e37;
  font-weight: 600;
}

.user-menu {
  margin-bottom: 2rem;
}

.user-profile {
  text-align: center;
  padding: 1.5rem;
  background-color: #f5f2ef;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
  margin-top: auto;
}

.profile-badge {
  margin-bottom: 1.5rem;
}

.username-link {
  text-decoration: none;
  color: inherit;
  display: block;
  transition: transform 0.2s ease;
  padding: 0.5rem;
  border-radius: 8px;
}

.username-link:hover {
  transform: translateY(-2px);
  background-color: rgba(111, 78, 55, 0.05);
}

.username {
  color: #4b3621;
  font-size: 1.4rem;
  font-weight: 600;
  margin: 0 0 0.5rem 0;
}

/* .user-role {
  display: inline-block;
  padding: 0.25rem 0.75rem;
  background-color: #6f4e37;
  color: white;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 500;
} */

.user-stats {
  display: flex;
  justify-content: space-around;
  padding-top: 1rem;
  border-top: 1px solid rgba(111, 78, 55, 0.1);
  margin-bottom: 1.5rem;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-decoration: none;
  color: inherit;
  transition: transform 0.2s ease;
  padding: 0.5rem;
  border-radius: 8px;
}

.stat-item:hover {
  transform: translateY(-2px);
  background-color: rgba(111, 78, 55, 0.05);
}

.stat-value {
  color: #6f4e37;
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 0.25rem;
}

.stat-label {
  color: #8b6b4a;
  font-size: 0.9rem;
}

.user-actions {
  padding-top: 1.5rem;
  border-top: 1px solid rgba(111, 78, 55, 0.1);
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.user-actions.no-border {
  margin-top: 0;
  padding-top: 0;
  border-top: none;
}

.user-actions .menu-item {
  width: 100%;
  text-align: center;
  padding: 0.75rem;
  border-radius: 8px;
  transition: all 0.2s ease;
  background-color: #eee5dd;
  color: #6f4e37;
  text-decoration: none;
  font-weight: 500;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.user-actions .menu-item:hover {
  transform: translateY(-2px);
  background-color: #e3d8ce;
}

.user-actions .menu-item.logout {
  background-color: #6f4e37;
  color: white;
}

.user-actions .menu-item.logout:hover {
  background-color: #8b6b4a;
}

.user-actions .menu-item.signup {
  background-color: #6f4e37;
  color: white;
}

.user-actions .menu-item.signup:hover {
  background-color: #8b6b4a;
}

.user-actions .menu-item i {
  font-size: 0.9rem;
}

.main-content {
  flex: 1;
  padding: 2rem;
  background-color: #faf7f5;
}

.content-section {
  width: 100%;
  margin: 0 auto;
  /* padding: 0 1rem; */
}

.content-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.section-title {
  color: #4b3621;
  font-size: 1.5rem;
  font-weight: 600;
  margin: 0;
  line-height: 1.5;
}

.study-tabs {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.tab-btn {
  padding: 0.3rem 1rem;
  border: 1px solid #d4c4b7;
  background-color: transparent;
  color: #4b3621;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 0.9rem;
}

.tab-btn:hover {
  background-color: #eee5dd;
  color: #6f4e37;
  transform: translateY(-2px);
  border-color: #c4b3a6;
}

.tab-btn.active {
  background-color: #6f4e37;
  color: white;
  border-color: #6f4e37;
  box-shadow: 0 2px 4px rgba(111, 78, 55, 0.2);
}

.profile-form-container {
  margin-top: 0;  /* Remove the top margin since we're using content-header's margin-bottom */
}

.profile-form {
  background-color: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group label {
  display: block;
  color: #4b3621;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.form-control {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 0.9rem;
  color: #4b3621;
}

.form-control:focus {
  border-color: #6f4e37;
  box-shadow: 0 0 0 0.2rem rgba(111, 78, 55, 0.25);
  outline: none;
}

.btn-primary {
  background-color: #6f4e37;
  border: none;
  color: white;
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.btn-primary:hover {
  background-color: #8b6b4a;
}

.study-list {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1rem;
  margin-bottom: 2rem;
  max-height: calc(100vh - 230px);
  overflow-y: auto;
  padding-right: 1rem;
  overscroll-behavior: contain;
  width: 100%;
}

@media (max-width: 1200px) {
  .study-list {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 992px) {
  .study-list {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .mypage-container {
    flex-direction: column;
  }

  .sidebar {
    width: 100%;
    border-right: none;
    border-bottom: 1px solid #eee5dd;
    padding: 1rem;
  }

  .main-content {
    padding: 1rem;
  }

  .content-section {
    padding: 0;
  }

  .study-tabs {
    overflow-x: auto;
    padding-bottom: 0.5rem;
    -webkit-overflow-scrolling: touch;
  }

  .tab-btn {
    white-space: nowrap;
  }

  .user-stats {
    flex-direction: column;
    gap: 1rem;
  }

  .stat-item {
    padding: 0.5rem;
    background-color: rgba(255, 255, 255, 0.5);
    border-radius: 8px;
  }
}

@media (max-width: 576px) {
  .study-list {
    grid-template-columns: 1fr;
  }
}

.study-list::-webkit-scrollbar {
  width: 8px;
}

.study-list::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 4px;
}

.study-list::-webkit-scrollbar-thumb {
  background: #6f4e37;
  border-radius: 4px;
}

.study-list::-webkit-scrollbar-thumb:hover {
  background: #8b6b4a;
}

.study-card {
  background-color: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  height: 100%;
}

.study-card:hover {
  transform: translateY(-4px);
}

.study-thumbnail {
  width: 100%;
  height: 160px;
  position: relative;
  overflow: hidden;
  flex-shrink: 0;
}

.study-thumbnail-skeleton {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #f5f5f5;
  overflow: hidden;
  z-index: 1;
}

.skeleton-content {
  width: 100%;
  height: 100%;
  background: linear-gradient(
    90deg,
    #f0f0f0 25%,
    #e0e0e0 50%,
    #f0f0f0 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
  0% {
    background-position: -200% 0;
  }
  100% {
    background-position: 200% 0;
  }
}

.study-thumbnail img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 2;
}

.study-info {
  padding: 0.75rem;
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 120px;
  position: relative;
}

.study-title {
  color: #4b3621;
  font-size: 1rem;
  font-weight: 600;
  margin: 0 0 0.5rem 0;
  line-height: 1.4;
}

.study-content {
  color: #666;
  font-size: 0.85rem;
  margin: 0 0 2.5rem 0;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  flex: 1;
  min-height: 2.4em;
  line-height: 1.4;
}

.study-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.8rem;
  color: #6f4e37;
  position: absolute;
  bottom: 0.75rem;
  left: 0.75rem;
  right: 0.75rem;
  padding-top: 0.5rem;
  border-top: 1px solid #eee5dd;
}

.study-status {
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 500;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.study-status.모집중 {
  background-color: #e3f2fd;
  color: #1976d2;
}

.study-status.모집완료 {
  background-color: #e8f5e9;
  color: #2e7d32;
}

.study-status.승인대기 {
  background-color: #fff3e0;
  color: #f57c00;
}

.study-status.승인 {
  background-color: #e8f5e9;
  color: #2e7d32;
}

.study-status.거절 {
  background-color: #ffebee;
  color: #c62828;
}
</style>
