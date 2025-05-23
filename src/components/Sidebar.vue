<template>
  <aside class="sidebar">
    <!-- 카테고리 목록 -->
    <div class="categories">
      <h5 class="sidebar-title">카테고리</h5>
      <ul class="category-list">
        <li 
          v-for="category in categories" 
          :key="category.id" 
          class="category-item"
          :class="{ 'selected': !isMypage && selectedCategory?.id === category.id && route.query.tab !== 'applied' && route.query.tab !== 'created' }"
        >
          <a href="#" @click.prevent="selectCategory(category)">{{ category.name }}</a>
        </li>
      </ul>
    </div>

    <!-- 사용자 메뉴 -->
    <div class="user-menu">
      <template v-if="!isLoggedIn">
        <div class="user-profile">
          <div class="user-actions no-border">
            <router-link to="/login" class="menu-item">로그인</router-link>
            <router-link to="/signup" class="menu-item signup">회원가입</router-link>
          </div>
        </div>
      </template>
    </div>

    <!-- 사용자 프로필 -->
    <div v-if="isLoggedIn" class="user-profile">
      <div class="profile-badge">
        <h3 class="username">{{ username }} 님</h3>
      </div>
      <div class="user-stats">
        <router-link to="/mypage?tab=applied" class="stat-item" :class="{ 'active': route.query.tab === 'applied' }">
          <span class="stat-value">{{ appliedStudies.length }}</span>
          <span class="stat-label">신청 스터디</span>
        </router-link>
        <router-link to="/mypage?tab=created" class="stat-item" :class="{ 'active': route.query.tab === 'created' }">
          <span class="stat-value">{{ createdStudies.length }}</span>
          <span class="stat-label">운영 스터디</span>
        </router-link>
      </div>
      <div class="user-actions">
        <router-link to="/mypage?tab=profile" class="menu-item">마이페이지 (내정보)</router-link>
        <a href="#" @click.prevent="logout" class="menu-item logout">로그아웃</a>
      </div>
    </div>
  </aside>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'

const props = defineProps({
  isMypage: {
    type: Boolean,
    default: false
  },
  selectedCategory: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['update:selectedCategory'])

const router = useRouter()
const route = useRoute()

// 상태 관리
const categories = ref([])
const isLoggedIn = ref(true)
const username = ref('홍길동')
const appliedStudies = ref([])
const createdStudies = ref([])

// 카테고리 데이터 가져오기
const fetchCategories = async () => {
  try {
    // TODO: 실제 API 호출로 대체
    // 임시 데이터
    categories.value = [
      { id: 1, name: '프로그래밍' },
      { id: 2, name: '디자인' },
      { id: 3, name: '마케팅' },
      { id: 4, name: '비즈니스' },
      { id: 5, name: '언어' }
    ]
    console.log('카테고리 데이터 로드됨:', categories.value)
  } catch (error) {
    console.error('카테고리 로딩 실패:', error)
  }
}

// 카테고리 선택 처리
const selectCategory = (category) => {
  console.log('Sidebar: Selecting category:', category) // 디버깅용 로그
  
  // 먼저 부모 컴포넌트에 카테고리 선택 알림
  emit('update:selectedCategory', category)
  
  if (props.isMypage) {
    // 마이페이지에서는 메인 페이지로 이동하면서 선택된 카테고리 정보를 쿼리 파라미터로 전달
    router.push({
      path: '/',
      query: { 
        category: category.id,
        categoryName: category.name 
      }
    })
  } else {
    // 메인 페이지에서도 URL에 카테고리 정보를 추가
    router.push({
      path: '/',
      query: { 
        category: category.id,
        categoryName: category.name 
      }
    })
  }
}

// 신청 스터디 클릭 처리
const handleAppliedStudyClick = (studyId) => {
  router.push({
    path: `/study/${studyId}`,
    query: { tab: 'applied' }
  })
}

// 로그아웃 함수
const logout = () => {
  // TODO: 로그아웃 로직 구현
  isLoggedIn.value = false
  username.value = ''
  router.push('/')
}

// 초기 데이터 로드
onMounted(() => {
  console.log('Sidebar 컴포넌트 마운트됨')
  fetchCategories()
  
  // 사용자 프로필 데이터 초기화
  if (isLoggedIn.value) {
    username.value = '홍길동'
    appliedStudies.value = [
      {
        id: 3,
        title: '알고리즘 스터디',
        content: '코딩 테스트 대비 알고리즘 문제 풀이',
        thumbnail: 'https://via.placeholder.com/150',
        currentMembers: 4,
        maxMembers: 6,
        applicationStatus: '승인대기'
      },
      {
        id: 4,
        title: '영어 회화 스터디',
        content: '매일 영어로 대화하며 실력 향상',
        thumbnail: 'https://via.placeholder.com/150',
        currentMembers: 3,
        maxMembers: 4,
        applicationStatus: '승인'
      },
      {
        id: 5,
        title: '마케팅 스터디',
        content: '디지털 마케팅 전략과 실전 사례',
        thumbnail: 'https://via.placeholder.com/150',
        currentMembers: 6,
        maxMembers: 6,
        applicationStatus: '거절'
      }
    ]
    createdStudies.value = [
      {
        id: 1,
        title: '프로그래밍 스터디',
        content: '함께 프로그래밍을 배우고 실력을 향상시켜요!',
        thumbnail: 'https://via.placeholder.com/150',
        currentMembers: 3,
        maxMembers: 5,
        status: '모집중'
      },
      {
        id: 2,
        title: '디자인 스터디',
        content: 'UI/UX 디자인을 함께 배워요',
        thumbnail: 'https://via.placeholder.com/150',
        currentMembers: 5,
        maxMembers: 5,
        status: '모집완료'
      }
    ]
  }
})
</script>

<style scoped>
.sidebar {
  width: 250px;
  background-color: #fbf9f8;
  padding: 2rem 1rem;
  border-right: 1px solid #eee5dd;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  min-height: calc(100vh - 60px);
  overflow-y: auto;
}

.sidebar-title {
  color: #6f4e37;
  font-weight: 600;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #eee5dd;
  text-align: center;
}

.categories {
  margin-bottom: 2rem;
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
  padding: 0.75rem;
  color: #4b3621;
  text-decoration: none;
  border-radius: 6px;
  transition: all 0.2s ease;
  font-size: 1rem;
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

.username-link.active,
.stat-item.active {
  background-color: #eee5dd;
  color: #6f4e37;
  font-weight: 600;
}

.username-link.active:hover,
.stat-item.active:hover {
  background-color: #eee5dd;
  transform: none;
}

@media (max-width: 768px) {
  .sidebar {
    width: 100%;
    border-right: none;
    border-bottom: 1px solid #eee5dd;
    padding: 1rem;
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
</style> 