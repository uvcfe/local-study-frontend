<template>
  <div class="main-container">
    <!-- 메인 콘텐츠 영역 -->
    <main class="main-content">
      <!-- 상단 영역 -->
      <div class="content-header">
        <h2 class="category-title">{{ selectedCategory?.name }} 스터디</h2>
        <div class="header-actions">
          <div class="location-selector">
            <select v-model="selectedSido" class="form-select" @change="handleSidoChange">
              <option value="">시/도 선택</option>
              <option v-for="sido in sidoList" :key="sido.id" :value="sido.id">{{ sido.name }}</option>
            </select>
            <h3 style="color: #6f4e37; font-size: 1rem; text-align: center; margin-top: 1px; margin-left: 4px;"> > </h3>
            <select v-model="selectedSigungu" class="form-select" @change="handleSigunguChange" :disabled="!selectedSido">
              <option value="">시/군/구 선택</option>
              <option v-for="sigungu in sigunguList" :key="sigungu.id" :value="sigungu.id">{{ sigungu.name }}</option>
            </select>
            <h3 style="color: #6f4e37; font-size: 1rem; text-align: center; margin-top: 1px; margin-left: 4px;"> > </h3>
            <select v-model="selectedDong" class="form-select" :disabled="!selectedSigungu">
              <option value="">읍/면/동 선택</option>
              <option v-for="dong in dongList" :key="dong.id" :value="dong.id">{{ dong.name }}</option>
            </select>
          </div>
        </div>
      </div>

      <!-- 스터디 목록 -->
      <div class="study-list" @wheel.prevent="handleStudyListScroll">
        <div v-for="study in filteredStudies" :key="study.id" class="study-card" @click="goToStudyDetail(study.id)">
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
              <span class="study-author">{{ study.author }}</span>
              <span class="study-members">{{ study.currentMembers }}/{{ study.maxMembers }}명</span>
            </div>
          </div>
        </div>

        <!-- 플로팅 스터디 만들기 버튼 -->
        <button 
          v-if="isLoggedIn" 
          class="floating-create-btn" 
          @click="goToCreateStudy"
          title="새 스터디 만들기"
        >
          <span class="btn-text">+</span>
        </button>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import logoImage from '@/assets/logo.png'
// import mockStudies from '@/data/mockStudies.json'
// import mockCategories from '@/data/mockCategories.json'
// import mockLocations from '@/data/mockLocations.json'
import axios from 'axios'

const router = useRouter()
const route = useRoute()
const categories = ref([])
const isLoggedIn = ref(true)
const username = ref('홍길동')
const selectedCategory = ref(null)
const searchQuery = ref('')

// 지역 선택 관련 상태
const selectedSido = ref('')
const selectedSigungu = ref('')
const selectedDong = ref('')
const sidoList = ref([])
const sigunguList = ref([])
const dongList = ref([])

// 지역 데이터 매핑
// const locationData = mockLocations.locationData

// 스터디 목록 관련 상태
const studies = ref([])
const currentPage = ref(1)

// 검색어와 문자열의 유사도를 계산하는 함수
const calculateSimilarity = (str1, str2) => {
  const s1 = str1.toLowerCase()
  const s2 = str2.toLowerCase()
  
  // 완전 일치
  if (s1 === s2) return 1
  
  // 부분 일치 (제목에서 검색어가 포함된 경우 더 높은 점수)
  if (s1.includes(s2)) return 0.9
  if (s2.includes(s1)) return 0.8
  
  // 단어 단위로 비교
  const words1 = s1.split(/\s+/)
  const words2 = s2.split(/\s+/)
  
  let matchCount = 0
  let totalWords = Math.max(words1.length, words2.length)
  
  for (const word1 of words1) {
    for (const word2 of words2) {
      // 단어가 정확히 일치하는 경우
      if (word1 === word2) {
        matchCount += 1
      } 
      // 한 단어가 다른 단어에 포함되는 경우
      else if (word1.includes(word2) || word2.includes(word1)) {
        matchCount += 0.8
      }
      // 단어의 일부가 일치하는 경우 (최소 2글자 이상)
      else if (word1.length > 1 && word2.length > 1) {
        let commonChars = 0
        for (let i = 0; i < Math.min(word1.length, word2.length); i++) {
          if (word1[i] === word2[i]) commonChars++
        }
        if (commonChars >= 2) {
          matchCount += 0.5 * (commonChars / Math.min(word1.length, word2.length))
        }
      }
    }
  }
  
  return matchCount / totalWords
}

// 검색어 처리 함수
const handleSearch = (query) => {
  console.log('Search query received:', query) // 디버깅용 로그
  searchQuery.value = query
  currentPage.value = 1 // 검색 시 첫 페이지로 이동
}

// 모든 필터 초기화
const resetAllFilters = () => {
  // 검색어 초기화
  searchQuery.value = ''
  // 지역 선택 초기화
  resetLocation()
  // 페이지 초기화
  currentPage.value = 1
}

// URL 쿼리 파라미터 처리
const processSearchQuery = () => {
  const query = route.query.search
  const categoryId = route.query.category
  const categoryName = route.query.categoryName

  if (query) {
    handleSearch(query)
  }

  if (categoryId) {
    const category = categories.value.find(cat => cat.id === parseInt(categoryId))
    if (category) {
      selectedCategory.value = category
    } else {
      // 카테고리를 찾지 못한 경우, 새로 생성
      selectedCategory.value = {
        id: parseInt(categoryId),
        name: categoryName || '카테고리'
      }
    }
  } else {
    // URL에 카테고리 정보가 없는 경우 기본 카테고리 선택
    selectedCategory.value = categories.value[0]
  }
}

// 라우트 변경 감지
watch(() => route.query.search, (newQuery) => {
  if (newQuery) {
    handleSearch(newQuery)
  } else {
    // 검색어가 제거된 경우 검색 초기화
    searchQuery.value = ''
    currentPage.value = 1
  }
})

// 리셋 이벤트 처리
const handleReset = () => {
  resetAllFilters()
}

// 스터디 만들기 페이지로 이동
const goToCreateStudy = () => {
  if (!isLoggedIn.value) {
    alert('로그인이 필요한 서비스입니다.')
    router.push('/login')
    return
  }
  // 현재 선택된 카테고리 정보를 저장
  if (selectedCategory.value) {
    localStorage.setItem('lastSelectedCategory', JSON.stringify({
      id: selectedCategory.value.id,
      name: selectedCategory.value.name
    }))
  }
  router.push('/create-study')
}

// 필터링된 스터디 목록
const filteredStudies = computed(() => {
  console.log('Selected Category:', selectedCategory.value) // 디버깅용 로그
  console.log('All Studies:', studies.value) // 디버깅용 로그
  
  if (!selectedCategory.value) return []
  
  // 카테고리 ID를 숫자로 변환하여 비교
  const selectedCategoryId = Number(selectedCategory.value.id)
  console.log('Selected Category ID:', selectedCategoryId) // 디버깅용 로그
  
  let filtered = studies.value.filter(study => {
    const studyCategoryId = Number(study.categoryId)
    console.log('Study Category ID:', studyCategoryId) // 디버깅용 로그
    return studyCategoryId === selectedCategoryId
  })
  
  console.log('Filtered Studies:', filtered) // 디버깅용 로그

  // 검색어가 있는 경우 필터링
  if (searchQuery.value) {
    const query = searchQuery.value.trim()
    if (query) {
      filtered = filtered.filter(study => {
        const titleSimilarity = calculateSimilarity(study.title, query)
        const contentSimilarity = calculateSimilarity(study.content, query)
        return Math.max(titleSimilarity, contentSimilarity) >= 0.3
      })
    }
  }

  // 지역 필터링
  if (selectedSido.value) {
    filtered = filtered.filter(study => {
      const matchesSido = study.location?.sido === selectedSido.value
      
      if (selectedSigungu.value) {
        const matchesSigungu = study.location?.sigungu === selectedSigungu.value
        
        if (selectedDong.value) {
          return matchesSido && matchesSigungu && study.location?.dong === selectedDong.value
        }
        
        return matchesSido && matchesSigungu
      }
      
      return matchesSido
    })
  }
  
  return filtered
})

// 카테고리 데이터 가져오기
const fetchCategories = async () => {
  try {
    // categories.value = mockCategories.categories
    const res = await axios.get('http://localhost:3000/category')
    categories.value = res.data.data
    // 기본 카테고리 선택
    if (categories.value.length > 0) {
      selectedCategory.value = categories.value[0]
    }
  } catch (error) {
    console.error('카테고리 로딩 실패:', error)
  }
}

// 카테고리 선택 처리
const selectCategory = (category) => {
  console.log('Mainpage: Selecting category:', category) // 디버깅용 로그
  selectedCategory.value = category
  resetLocation() // 카테고리 변경 시 지역 선택 초기화
  
  // URL 업데이트
  router.push({
    path: '/',
    query: { 
      category: category.id,
      categoryName: category.name 
    }
  })
}

// 라우트 변경 감지
watch(() => route.query.category, (newCategoryId) => {
  if (newCategoryId) {
    const category = categories.value.find(cat => cat.id === Number(newCategoryId))
    if (category) {
      selectedCategory.value = category
    }
  }
}, { immediate: true })

// 로그아웃 처리
const logout = () => {
  // TODO: 로그아웃 로직 구현
  isLoggedIn.value = false
  username.value = ''
  router.push('/')
}

// 로그인 상태 확인
const checkLoginStatus = () => {
  // TODO: 실제 로그인 상태 확인 로직 구현
  // 임시로 로그인 상태 체크
  const token = localStorage.getItem('token')
  if (token) {
    isLoggedIn.value = true
    username.value = '사용자' // TODO: 실제 사용자 이름으로 대체
  }
}

// 지역 선택 핸들러
const handleSidoChange = async () => {
  selectedSigungu.value = ''
  selectedDong.value = ''
  sigunguList.value = []
  dongList.value = []
  if (selectedSido.value) {
    await fetchSigunguList(selectedSido.value)
  }
}

const handleSigunguChange = async () => {
  selectedDong.value = ''
  dongList.value = []
  if (selectedSido.value && selectedSigungu.value) {
    await fetchDongList(selectedSigungu.value)
  }
}

// 지역 데이터 가져오기
const fetchSidoList = async () => {
  try {
    const res = await axios.get('http://localhost:3000/city')
    sidoList.value = res.data.data
  } catch (e) {
    console.error('시/도 목록 불러오기 실패', e)
  }
}

const fetchSigunguList = async (cityId) => {
  try {
    const res = await axios.get(`http://localhost:3000/district/${cityId}`)
    sigunguList.value = res.data.data
  } catch (e) {
    console.error('시/군/구 목록 불러오기 실패', e)
  }
}

const fetchDongList = async (districtId) => {
  try {
    const res = await axios.get(`http://localhost:3000/town/${districtId}`)
    dongList.value = res.data.data
  } catch (e) {
    console.error('읍/면/동 목록 불러오기 실패', e)
  }
}

// 스터디 목록 가져오기
const fetchStudies = async () => {
  try {
    console.log('Fetching studies...') // 디버깅용 로그
    // TODO: 실제 API 호출로 대체
    studies.value = mockStudies.studies.map(study => ({
      ...study,
      isImageLoading: true
    }))
    console.log('Studies fetched:', studies.value) // 디버깅용 로그
  } catch (error) {
    console.error('스터디 목록 로딩 실패:', error)
  }
}

// 스터디 상세 페이지로 이동
const goToStudyDetail = (studyId) => {
  router.push(`/study/${studyId}`)
}

// 스터디 목록 스크롤 처리
const handleStudyListScroll = (event) => {
  const studyList = event.currentTarget
  const { scrollTop, scrollHeight, clientHeight } = studyList
  
  // 스크롤이 맨 위에 있고 위로 스크롤하려고 할 때
  if (scrollTop === 0 && event.deltaY < 0) {
    return
  }
  
  // 스크롤이 맨 아래에 있고 아래로 스크롤하려고 할 때
  if (scrollTop + clientHeight >= scrollHeight && event.deltaY > 0) {
    return
  }
  
  // 그 외의 경우 스터디 목록 스크롤 허용
  studyList.scrollTop += event.deltaY
}

// 지역 선택 초기화
const resetLocation = () => {
  selectedSido.value = ''
  selectedSigungu.value = ''
  selectedDong.value = ''
  sigunguList.value = []
  dongList.value = []
}

const appliedStudies = ref([])
const createdStudies = ref([])

// 초기 데이터 로드
onMounted(() => {
  fetchCategories()
  checkLoginStatus()
  fetchStudies()
  fetchSidoList() // 시/도 목록 가져오기 추가
  processSearchQuery()
  
  // 임시 데이터
  appliedStudies.value = [
    {
      id: 3,
      title: '알고리즘 스터디',
      content: '코딩 테스트 대비 알고리즘 문제 풀이',
      thumbnail: 'https://via.placeholder.com/150',
      currentMembers: 4,
      maxMembers: 6,
      applicationStatus: '승인대기'
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
    }
  ]

  // Add preload link for logo image
  const link = document.createElement('link')
  link.rel = 'preload'
  link.as = 'image'
  link.href = logoImage
  document.head.appendChild(link)
})

// 이미지 로드 핸들러
const handleImageLoad = (study) => {
  study.isImageLoading = false
}

// 이미지 에러 핸들러
const handleImageError = (study) => {
  study.isImageLoading = false
}

// 컴포넌트 메서드 노출
defineExpose({
  handleSearch,
  handleReset
})
</script>

<style scoped>
.main-container {
  display: flex;
  min-height: calc(100vh - 60px); /* 헤더 높이 제외 */
  background-color: #faf7f5;
}

.sidebar {
  width: 250px;
  background-color: #fbf9f8;
  padding: 2rem 1rem;
  border-right: 1px solid #eee5dd;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
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

.user-actions {
  margin-top: 1.5rem;
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

.main-content {
  flex: 1;
  padding: 2rem;
  background-color: #faf7f5;
}

.content-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.category-title {
  color: #4b3621;
  font-size: 1.5rem;
  font-weight: 600;
  margin: 0;
  line-height: 1.5;
}

.header-actions {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.location-selector {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.form-select {
  padding: 0.4rem;
  border-radius: 6px;
  border: 1px solid #ddd;
  color: #4b3621;
  background-color: white;
  min-width: 110px;
  font-size: 0.8rem;
}

.form-select:focus {
  border-color: #6f4e37;
  box-shadow: 0 0 0 0.2rem rgba(111, 78, 55, 0.25);
}

h3 {
  color: #6f4e37;
  font-size: 1.2rem;
  margin: 0;
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
  .mainpage-container {
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
  border-radius: 8px 8px 0 0;
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

.pagination,
.page-btn,
.page-info {
  display: none;
}

.create-study-section {
  margin-bottom: 2rem;
  padding: 0 0.5rem;
}

.create-study-btn {
  width: 100%;
  padding: 0.875rem;
  border: none;
  border-radius: 8px;
  background-color: #6f4e37;
  color: white;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  font-size: 0.95rem;
  font-weight: 500;
  box-shadow: 0 2px 8px rgba(111, 78, 55, 0.15);
}

.create-study-btn:hover {
  background-color: #8b6b4a;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(111, 78, 55, 0.2);
}

.create-study-btn i {
  font-size: 0.9rem;
}

.categories {
  flex: 1;
}

/* 플로팅 버튼 스타일 */
.floating-create-btn {
  position: fixed;
  bottom: 5rem;
  right: 5rem;
  width: 64px;
  height: 64px;
  border: none;
  border-radius: 50%;
  background-color: #6f4e37;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 12px rgba(111, 78, 55, 0.2);
  z-index: 1000;
  font-size: 1.75rem;
  font-weight: 500;
}

.floating-create-btn .btn-text {
  font-size: 2rem;
  line-height: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: -2px; /* + 기호를 정확히 중앙에 위치시키기 위한 조정 */
}

.floating-create-btn:hover {
  background-color: #8b6b4a;
  transform: translateY(-2px) scale(1.05);
  box-shadow: 0 6px 16px rgba(111, 78, 55, 0.3);
}

.floating-create-btn:active {
  transform: translateY(0) scale(0.95);
}

/* 반응형 디자인 */
@media (max-width: 768px) {
  .floating-create-btn {
    width: 56px;
    height: 56px;
    font-size: 1.5rem;
  }

  .floating-create-btn .btn-text {
    font-size: 1.75rem;
  }
}

.profile-badge {
  margin-bottom: 1.5rem;
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
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
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
</style>
