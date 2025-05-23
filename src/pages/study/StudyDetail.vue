<template>
  <div class="study-detail-container">
    <!-- 메인 콘텐츠 영역 -->
    <main class="main-content">
      <!-- 상단 영역 -->
      <div class="content-header">
        <h2 class="category-title">{{ selectedCategory?.name }} 스터디</h2>
      </div>

      <!-- 스터디 상세 정보 -->
      <div class="study-detail">
        <!-- 좌측 영역 -->
        <div class="left-section">
          <!-- 썸네일 영역 -->
          <div class="thumbnail-section">
            <template v-if="isEditing">
              <div class="thumbnail-wrapper" style="position: relative; width: 100%; height: 100%;">
                <template v-if="editedStudy.thumbnail && !thumbnailDeleted">
                  <img :src="editedStudy.thumbnail" alt="썸네일 미리보기" class="study-thumbnail">
                  <button v-if="editedStudy.thumbnail" class="delete-thumbnail" @click.stop="deleteThumbnail" style="position: absolute; top: 8px; right: 8px; width: 24px; height: 24px; border-radius: 50%; background-color: #eee5dd; border: 1px solid #e3d8ce; color: #6f4e37; display: flex; align-items: center; justify-content: center; cursor: pointer; z-index: 1; font-size: 1.2rem; font-weight: 700; line-height: 1; padding: 0;">×</button>
                </template>
                <template v-else>
                  <div class="thumbnail-upload-empty" @click="triggerFileInput" style="width: 100%; height: 100%; display: flex; align-items: center; justify-content: center; color: #8b6b4a; font-size: 1rem; background: #faf7f5; border-radius: 8px; cursor: pointer; border: 1.5px dashed #e3d8ce;">사진 등록</div>
                </template>
                <input type="file" ref="fileInput" @change="handleThumbnailChange" accept="image/*" style="display: none">
              </div>
            </template>
            <template v-else>
              <img :src="study.thumbnail || logoImage" :alt="study.title" class="study-thumbnail" loading="lazy" decoding="async" fetchpriority="high" width="800" height="480" sizes="(max-width: 768px) 100vw, 50vw">
            </template>
          </div>
          <!-- 참여자 목록 -->
          <div class="participants-section">
            <div class="participants-header">
              <h3>참여자 목록</h3>
              <span class="participants-count">{{ study.participants?.length || 0 }}/{{ study.maxMembers }}명</span>
            </div>
            <ul class="participants-list">
              <li v-for="participant in study.participants" :key="participant.id" class="participant-item">
                <div class="name-role">
                  <span class="participant-name">{{ participant.name }}</span>
                  <span class="participant-role" v-if="participant.isAuthor">👑</span>
                </div>
                <button v-if="isAuthor && !participant.isAuthor && isEditing" class="kick-btn" @click="kickParticipant(participant)">추방</button>
              </li>
            </ul>
          </div>
        </div>

        <!-- 우측 영역 -->
        <div class="right-section">
          <div class="form-group title-category-group">
            <div class="content-title">
              <template v-if="!isEditing">
                {{ study.title }}
              </template>
              <input
                v-else
                v-model="editedStudy.title"
                type="text"
                class="title-input"
                placeholder="스터디 제목을 입력하세요"
              >
            </div>
            <div class="category-members-group">
              <div class="content-category">
                <template v-if="!isEditing">
                  {{ selectedCategory?.name }}
                </template>
                <select
                  v-else
                  v-model="editedStudy.category_id"
                  class="form-select"
                >
                  <option value="" disabled>카테고리 선택</option>
                  <option v-for="category in categories" :key="category.id" :value="category.id">
                    {{ category.name }}
                  </option>
                </select>
              </div>
              <div class="content-members">
                <span class="info-label">총 인원</span>
                <span class="info-content">
                  <i class="fas fa-users"></i>
                  <template v-if="!isEditing">
                    {{ study.maxMembers }}명
                  </template>
                  <input
                    v-else
                    v-model.number="editedStudy.maxMembers"
                    type="number"
                    class="number-input"
                    min="2"
                    max="20"
                  >
                </span>
              </div>
            </div>
          </div>
          <div class="form-group">
            <div class="content-location">
              <span class="info-label">지역</span>
              <span class="info-content">
                <i class="fas fa-map-marker-alt"></i>
                <template v-if="!isEditing">
                  {{ study.location?.sido }} {{ study.location?.sigungu }} {{ study.location?.dong }}
                </template>
                <div v-else class="location-dropdowns">
                  <select v-model="editedStudy.sido" @change="handleSidoChange" class="form-select" required>
                    <option value="">시/도 선택</option>
                    <option v-for="sido in sidoList" :key="sido" :value="sido">{{ sido }}</option>
                  </select>
                  <select v-model="editedStudy.sigungu" @change="handleSigunguChange" class="form-select" :disabled="!editedStudy.sido" required>
                    <option value="">시/군/구 선택</option>
                    <option v-for="sigungu in sigunguList" :key="sigungu" :value="sigungu">{{ sigungu }}</option>
                  </select>
                  <select v-model="editedStudy.dong" class="form-select" :disabled="!editedStudy.sigungu" required>
                    <option value="">읍/면/동 선택</option>
                    <option v-for="dong in dongList" :key="dong" :value="dong">{{ dong }}</option>
                  </select>
                </div>
              </span>
            </div>
          </div>
          <div class="form-group">
            <div class="content-date">
              <span class="info-label">기간</span>
              <span class="info-content">
                <i class="fas fa-calendar-alt"></i>
                <template v-if="!isEditing">
                  {{ formatDate(study.startDate) }} ~ {{ formatDate(study.endDate) }}
                </template>
                <div v-else class="date-inputs">
                  <div class="date-picker-wrapper" @click="focusDateInput('startDate')">
                    <input 
                      type="date" 
                      id="startDate" 
                      v-model="editedStudy.startDate" 
                      required
                      class="date-input"
                      :min="getTodayDate()"
                      @change="validateDates"
                    >
                    <div class="date-display">
                      {{ formatDate(editedStudy.startDate) }}
                    </div>
                  </div>
                  <span class="date-separator">~</span>
                  <div class="date-picker-wrapper" @click="focusDateInput('endDate')">
                    <input 
                      type="date" 
                      id="endDate" 
                      v-model="editedStudy.endDate" 
                      required
                      class="date-input"
                      :min="editedStudy.startDate || getTodayDate()"
                      @change="validateDates"
                    >
                    <div class="date-display">
                      {{ formatDate(editedStudy.endDate) }}
                    </div>
                  </div>
                </div>
              </span>
            </div>
          </div>
          <div class="form-group">
            <div class="content-text">
              <template v-if="!isEditing">
                {{ study.content }}
              </template>
              <textarea
                v-else
                v-model="editedStudy.content"
                class="form-textarea"
                placeholder="스터디에 대한 설명을 입력하세요"
              ></textarea>
            </div>
          </div>
          <div class="form-actions">
            <template v-if="isAuthor">
              <template v-if="!isEditing">
                <button type="button" class="edit-btn" @click="startEditing">수정하기</button>
                <button type="button" class="delete-btn" @click="handleDeleteStudy">삭제하기</button>
              </template>
              <template v-else>
                <button type="button" class="cancel-btn" @click="cancelEditing">취소</button>
                <button type="button" class="submit-btn" @click="handleUpdateStudy">저장하기</button>
              </template>
            </template>
            <template v-else>
              <button 
                v-if="isLoggedIn && !isParticipant" 
                class="join-btn"
                @click="handleJoinStudy"
                :disabled="study.currentMembers >= study.maxMembers"
              >
                참가 신청
              </button>
              <button 
                v-else-if="isLoggedIn && isParticipant" 
                class="leave-btn"
                @click="handleLeaveStudy"
              >
                참가 취소
              </button>
              <button 
                v-else 
                class="login-btn"
                @click="goToLogin"
              >
                로그인하고 참가하기
              </button>
            </template>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import logoImage from '@/assets/logo.png'
import mockStudies from '@/data/mockStudies.json'
import mockCategories from '@/data/mockCategories.json'
import mockLocations from '@/data/mockLocations.json'
import mockAppliedStudies from '@/data/mockAppliedStudies.json'

const router = useRouter()
const route = useRoute()
const study = ref({})
const categories = ref([])
const isLoggedIn = ref(true)
const username = ref('')
const selectedCategory = ref(null)
const isParticipant = ref(false)
const isAuthor = ref(false)
const selectedSido = ref('')
const selectedSigungu = ref('')
const selectedDong = ref('')
const sigunguList = ref([])
const dongList = ref([])
const isEditing = ref(false)
const editedStudy = ref({
  title: '',
  category_id: '',
  maxMembers: 0,
  startDate: '',
  endDate: '',
  content: '',
  sido: '',
  sigungu: '',
  dong: ''
})
const appliedStudies = ref([])
const createdStudies = ref([])
const sidoList = ref([])
const originalThumbnail = ref('')
const thumbnailDeleted = ref(false)
const fileInput = ref(null)
const originalParticipants = ref([])

// 날짜 포맷팅 함수
const formatDate = (dateString) => {
  if (!dateString) return ''
  const date = new Date(dateString)
  return date.toLocaleDateString('ko-KR', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

// 지역 데이터 매핑
const locationData = mockLocations.locationData

// 지역 선택 핸들러
const handleSidoChange = () => {
  selectedSigungu.value = ''
  selectedDong.value = ''
  sigunguList.value = selectedSido.value ? Object.keys(locationData[selectedSido.value] || {}) : []
}

const handleSigunguChange = () => {
  selectedDong.value = ''
  dongList.value = selectedSido.value && selectedSigungu.value 
    ? (locationData[selectedSido.value]?.[selectedSigungu.value] || [])
    : []
}

// 스터디 상세 정보 가져오기
const fetchStudyDetail = async () => {
  try {
    const studyId = parseInt(route.params.id)
    const isAppliedStudy = route.query.tab === 'applied'
    const isCreatedStudy = route.query.tab === 'created'
    
    // 신청 스터디인 경우
    if (isAppliedStudy) {
      const foundStudy = mockAppliedStudies.appliedStudies.find(s => s.id === studyId)
      if (foundStudy) {
        study.value = foundStudy
        // 작성자 여부 확인
        isAuthor.value = false
        // 신청 상태에 따라 참여자 여부 설정
        isParticipant.value = foundStudy.applicationStatus === '승인'
        
        // 지역 선택 초기화
        selectedSido.value = study.value.location.sido
        handleSidoChange()
        selectedSigungu.value = study.value.location.sigungu
        handleSigunguChange()
        selectedDong.value = study.value.location.dong

        // 카테고리 선택
        const category = categories.value.find(cat => cat.id === study.value.categoryId)
        if (category) {
          selectedCategory.value = category
        }
        return
      }
    }
    
    // 운영 스터디인 경우
    if (isCreatedStudy) {
      const foundStudy = mockStudies.studies.find(s => s.id === studyId)
      if (foundStudy) {
        study.value = foundStudy
        // 작성자 여부 확인
        isAuthor.value = true
        
        // 지역 선택 초기화
        selectedSido.value = study.value.location.sido
        handleSidoChange()
        selectedSigungu.value = study.value.location.sigungu
        handleSigunguChange()
        selectedDong.value = study.value.location.dong

        // 카테고리 선택
        const category = categories.value.find(cat => cat.id === study.value.categoryId)
        if (category) {
          selectedCategory.value = category
        }
        return
      }
    }
    
    // 일반 스터디인 경우
    const foundStudy = mockStudies.studies.find(s => s.id === studyId)
    if (!foundStudy) {
      console.error('스터디를 찾을 수 없습니다:', studyId)
      return
    }

    study.value = foundStudy
    // 작성자 여부 확인 (임시로 true로 설정)
    isAuthor.value = true

    // 지역 선택 초기화
    if (isAuthor.value) {
      selectedSido.value = study.value.location.sido
      handleSidoChange()
      selectedSigungu.value = study.value.location.sigungu
      handleSigunguChange()
      selectedDong.value = study.value.location.dong
    }

    // 카테고리 선택
    const category = categories.value.find(cat => cat.id === study.value.categoryId)
    if (category) {
      selectedCategory.value = category
    }
  } catch (error) {
    console.error('스터디 상세 정보 로딩 실패:', error)
  }
}

// 카테고리 데이터 가져오기
const fetchCategories = async () => {
  try {
    // TODO: 실제 API 호출로 대체
    categories.value = mockCategories.categories
    
    // 스터디 정보를 가져온 후 카테고리 선택
    await fetchStudyDetail()
  } catch (error) {
    console.error('카테고리 로딩 실패:', error)
  }
}

// 카테고리 선택 처리
const selectCategory = (category) => {
  // 메인 페이지로 이동하면서 선택된 카테고리 정보를 쿼리 파라미터로 전달
  router.push({
    path: '/',
    query: { 
      category: category.id,
      categoryName: category.name 
    }
  })
}

// 로그인 상태 확인
const checkLoginStatus = () => {
  // TODO: 실제 로그인 상태 확인 로직 구현
  const token = localStorage.getItem('token')
  if (token) {
    isLoggedIn.value = true
    username.value = '사용자'
  }
}

// 로그아웃 처리
const logout = () => {
  // TODO: 로그아웃 로직 구현
  isLoggedIn.value = false
  username.value = ''
  router.push('/')
}

// 스터디 참가 처리
const handleJoinStudy = async () => {
  if (!isLoggedIn.value) {
    router.push('/login')
    return
  }
  
  try {
    // TODO: 실제 API 호출로 대체
    isParticipant.value = true
    study.value.currentMembers++
    alert('스터디 참가 신청이 완료되었습니다.')
  } catch (error) {
    console.error('스터디 참가 실패:', error)
  }
}

// 스터디 참가 취소 처리
const handleLeaveStudy = async () => {
  try {
    // TODO: 실제 API 호출로 대체
    isParticipant.value = false
    study.value.currentMembers--
    alert('스터디 참가가 취소되었습니다.')
  } catch (error) {
    console.error('스터디 참가 취소 실패:', error)
  }
}

// 로그인 페이지로 이동
const goToLogin = () => {
  router.push('/login')
}

// 수정 시작
const startEditing = () => {
  editedStudy.value = {
    title: study.value.title,
    category_id: study.value.category_id,
    maxMembers: study.value.maxMembers,
    startDate: study.value.startDate,
    endDate: study.value.endDate,
    content: study.value.content,
    sido: study.value.location?.sido || '',
    sigungu: study.value.location?.sigungu || '',
    dong: study.value.location?.dong || '',
    thumbnail: study.value.thumbnail
  }
  originalThumbnail.value = study.value.thumbnail
  thumbnailDeleted.value = false
  originalParticipants.value = JSON.parse(JSON.stringify(study.value.participants))
  sidoList.value = Object.keys(locationData)
  isEditing.value = true
}

// 수정 취소
const cancelEditing = () => {
  isEditing.value = false
  editedStudy.value = {
    title: study.value.title,
    category_id: study.value.category_id,
    maxMembers: study.value.maxMembers,
    startDate: study.value.startDate,
    endDate: study.value.endDate,
    content: study.value.content,
    sido: study.value.location?.sido || '',
    sigungu: study.value.location?.sigungu || '',
    dong: study.value.location?.dong || '',
    thumbnail: originalThumbnail.value
  }
  thumbnailDeleted.value = false
  study.value.participants = JSON.parse(JSON.stringify(originalParticipants.value))
}

// 스터디 삭제 처리
const handleDeleteStudy = async () => {
  if (!confirm('정말로 이 스터디를 삭제하시겠습니까?')) {
    return
  }
  
  try {
    // TODO: 실제 API 호출로 대체
    // 임시 데이터 삭제 처리
    alert('스터디가 삭제되었습니다.')
    router.push('/')
  } catch (error) {
    console.error('스터디 삭제 실패:', error)
    alert('스터디 삭제에 실패했습니다.')
  }
}

// 스터디 정보 업데이트
const handleUpdateStudy = async () => {
  try {
    // TODO: 실제 API 호출로 대체
    study.value.title = editedStudy.value.title
    study.value.category_id = editedStudy.value.category_id
    study.value.maxMembers = editedStudy.value.maxMembers
    study.value.startDate = editedStudy.value.startDate
    study.value.endDate = editedStudy.value.endDate
    study.value.content = editedStudy.value.content
    study.value.location = {
      sido: editedStudy.value.sido,
      sigungu: editedStudy.value.sigungu,
      dong: editedStudy.value.dong
    }
    isEditing.value = false
    alert('스터디 정보가 수정되었습니다.')
  } catch (error) {
    console.error('스터디 수정 실패:', error)
    alert('스터디 수정에 실패했습니다.')
  }
}

// 날짜 입력 필드 포커스 함수 추가
const focusDateInput = (inputId) => {
  const input = document.getElementById(inputId)
  if (input) {
    input.showPicker()
  }
}

// 날짜 유효성 검사
const validateDates = () => {
  if (editedStudy.value.startDate && editedStudy.value.endDate) {
    const startDate = new Date(editedStudy.value.startDate)
    const endDate = new Date(editedStudy.value.endDate)
    
    if (endDate < startDate) {
      alert('종료일은 시작일보다 이후여야 합니다.')
      editedStudy.value.endDate = editedStudy.value.startDate
    }
  }
}

// 오늘 날짜를 YYYY-MM-DD 형식으로 반환하는 함수
const getTodayDate = () => {
  const today = new Date()
  const year = today.getFullYear()
  const month = String(today.getMonth() + 1).padStart(2, '0')
  const day = String(today.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

// Add preload link for logo image
onMounted(() => {
  const link = document.createElement('link')
  link.rel = 'preload'
  link.as = 'image'
  link.href = logoImage
  document.head.appendChild(link)
})

onMounted(() => {
  sidoList.value = Object.keys(locationData)
  fetchCategories()
  checkLoginStatus()
  fetchStudyDetail()
  
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
})

const handleThumbnailChange = (e) => {
  const file = e.target.files[0]
  if (file) {
    const reader = new FileReader()
    reader.onload = (event) => {
      editedStudy.value.thumbnail = event.target.result
      thumbnailDeleted.value = false
    }
    reader.readAsDataURL(file)
  }
}

const triggerFileInput = () => {
  if (fileInput.value) fileInput.value.click()
}

const deleteThumbnail = () => {
  editedStudy.value.thumbnail = ''
  thumbnailDeleted.value = true
}

const kickParticipant = (participant) => {
  if (confirm(`${participant.name}님을 추방하시겠습니까?`)) {
    // TODO: 실제 추방 API 연동
    study.value.participants = study.value.participants.filter(p => p.id !== participant.id)
    alert(`${participant.name}님이 추방되었습니다.`)
  }
}
</script>

<style scoped>
.study-detail-container {
  display: flex;
  min-height: calc(100vh - 60px);
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

.main-content {
  flex: 1;
  padding: 2rem;
  background-color: #faf7f5;
}

.content-header {
  margin-bottom: 2rem;
}

.category-title {
  color: #4b3621;
  font-size: 1.5rem;
  font-weight: 600;
  margin: 0;
}

.study-detail {
  display: flex;
  gap: 2rem;
  background-color: white;
  border-radius: 12px;
  padding: 2rem 2rem 1rem 2rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  height: calc(100vh - 220px);
  min-height: 600px;
  overflow: hidden;
}

.left-section {
  width: 300px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  gap: 2rem;
  position: relative;
}

.thumbnail-section {
  width: 100%;
  aspect-ratio: 4/4;
  overflow: hidden;
  border-radius: 8px;
}

.study-thumbnail {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.participants-section {
  background-color: #fbf9f8;
  border-radius: 8px;
  padding: 1.5rem;
  border: 1px solid #eee5dd;
  position: absolute;
  bottom: 1rem;
  left: 0;
  right: 0;
  margin-bottom: 0.25rem;
}

.participants-section .participants-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid #eee5dd;
}

.participants-section .participants-header h3 {
  color: #6f4e37;
  font-size: 1.2rem;
  margin: 0;
}

.participants-section .participants-count {
  color: #8b6b4a;
  font-size: 0.9rem;
  font-weight: 600;
  background-color: #f5f2ef;
  padding: 0.25rem 0.75rem;
  border-radius: 12px;
  border: 1px solid #eee5dd;
}

.participants-list {
  list-style: none;
  padding: 0;
  margin: 0;
  max-height: 130px;
  overflow-y: auto;
}

.participant-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.5rem;
  transition: background 0.2s;
}

.participant-item:hover {
  background: #f5f2ef;
}

.name-role {
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

.participant-role {
  color: #6f4e37;
  font-size: 0.9rem;
}

.right-section {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.form-group {
  margin-bottom: 0.25rem;
  flex-shrink: 0;
  overflow: hidden;
}

.form-group:has(.form-textarea) {
  border: none;
  padding: 0;
  background-color: transparent;
}

.form-group label {
  display: block;
  margin-bottom: 0.25rem;
  color: #4b3621;
  font-weight: 600;
  font-size: 0.95rem;
}

.form-input {
  width: 100%;
  padding: 0.5rem 0.75rem;
  border: 1px solid #e3d8ce;
  border-radius: 10px;
  font-size: 0.9rem;
  color: #4b3621;
  background-color: #fff;
  transition: all 0.2s ease;
}

.form-input:hover {
  border-color: #c4b5a5;
  background-color: #fdfbf9;
}

.form-input:focus {
  outline: none;
  border-color: #6f4e37;
  box-shadow: 0 0 0 2px rgba(111, 78, 55, 0.1);
  background-color: #fff;
}

.form-textarea {
  width: 100%;
  padding: 0.75rem;
  border: none;
  border-radius: 8px;
  font-size: 1.1rem;
  line-height: 1.6;
  color: #4b3621;
  background-color: transparent;
  transition: all 0.2s ease;
  height: 300px;
  resize: none;
  overflow-y: auto;
  margin-bottom: 0.25rem;
}

.form-textarea:hover {
  background-color: #fdfbf9;
}

.form-textarea:focus {
  outline: none;
  background-color: #fff;
}

.form-row {
  display: flex;
  gap: 1rem;
  margin-bottom: 0.75rem;
  flex-shrink: 0;
}

.form-row .form-group {
  flex: 1;
  margin-bottom: 0;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 1rem;
  margin-top: 0.5rem;
  padding-top: 0.5rem;
  border-top: 1px solid #e3d8ce;
  flex-shrink: 0;
}

.cancel-btn {
  padding: 0.75rem 1.5rem;
  background-color: #eee5dd;
  color: #6f4e37;
  border: 1px solid #e3d8ce;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-weight: 600;
}

.cancel-btn:hover {
  background-color: #eee5dd;
  border-color: #c4b5a5;
}

.submit-btn {
  padding: 0.75rem 1.5rem;
  background-color: #8b6b4a;
  color: white;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  transition: background-color 0.2s ease;
  font-weight: 600;
}

.submit-btn:hover {
  background-color: #5a3f2e;
}

.location-selector {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 0.75rem;
  flex-shrink: 0;
}

.location-dropdowns {
  display: flex;
  gap: 0.75rem;
  margin-bottom: 0;
  flex: 1;
  max-width: 600px;
}

.form-select {
  width: 100%;
  height: 100%;
  padding: 0 0.5rem;
  border: none;
  background: transparent;
  color: #6f4e37;
  font-size: 0.9rem;
  font-weight: 600;
  border-radius: 20px;
  text-align: center;
  box-sizing: border-box;
  appearance: none;
  display: flex;
  align-items: center;
  justify-content: center;
}

.form-select option {
  text-align: center;
}

.form-select:focus {
  outline: none;
  border-color: #6f4e37;
  box-shadow: 0 0 0 2px rgba(111, 78, 55, 0.1);
}

.form-select:disabled {
  background-color: #f5f2ef;
  cursor: not-allowed;
  opacity: 0.7;
}

.date-input {
  width: 100%;
  padding: 0.4rem;
  border: 1px solid #e3d8ce;
  border-radius: 6px;
  font-size: 0.9rem;
  color: #4b3621;
  background-color: #fff;
  cursor: pointer;
  transition: all 0.2s ease;
  height: 36px;
}

.date-input:hover {
  border-color: #c4b5a5;
  background-color: #fdfbf9;
}

.date-input:focus {
  outline: none;
  border-color: #6f4e37;
  box-shadow: 0 0 0 2px rgba(111, 78, 55, 0.1);
}

.number-input {
  width: 80px;
  padding: 0.4rem;
  border: 1px solid #e3d8ce;
  border-radius: 6px;
  font-size: 0.9rem;
  color: #4b3621;
  background-color: #fff;
  text-align: center;
  transition: all 0.2s ease;
  height: 36px;
}

.number-input:hover {
  border-color: #c4b5a5;
  background-color: #fdfbf9;
}

.number-input:focus {
  outline: none;
  border-color: #6f4e37;
  box-shadow: 0 0 0 2px rgba(111, 78, 55, 0.1);
}

@media (max-width: 768px) {
  .form-group {
    margin-bottom: 0.5rem;
  }
  .form-row {
    gap: 0.75rem;
  }
  .form-actions {
    margin-top: 1.25rem;
    padding-top: 1.25rem;
  }
  .location-dropdowns {
    flex-direction: column;
    gap: 0.75rem;
  }
  .number-input {
    width: 70px;
    font-size: 0.85rem;
  }
  .form-select {
    padding: 0.6rem 2.25rem 0.6rem 0.6rem;
    font-size: 0.85rem;
    background-size: 0.9rem;
    height: 38px;
    min-height: 38px;
  }
}

.title-category-group {
  display: flex;
  align-items: flex-start;
  gap: 1rem;
  margin-bottom: 0.75rem;
  flex-shrink: 0;
  width: 100%;
  border-bottom: 1px solid #eee5dd;
  padding-bottom: 1rem;
}

.title-group {
  flex: 1;
  max-width: 50%;
}

.category-members-group {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 0.7rem;
  min-width: 0;
  flex: 1 1 0;
  justify-content: flex-end;
  margin-left: auto;
}

.content-category {
  display: inline-block;
  padding: 0.5rem 0.7rem;
  background-color: #eee5dd;
  color: #6f4e37;
  border-radius: 20px;
  font-size: 0.87rem;
  font-weight: 600;
  white-space: nowrap;
  text-align: center;
  min-width: 120px;
  height: 36px;
  box-sizing: border-box;
}

.content-members {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  min-width: 120px;
  font-size: 0.87rem;
  color: #6f4e37;
  padding: 0.5rem 0.5rem 0.5rem 0.5rem;
  background-color: #f5f2ef;
  border-radius: 20px;
  border: 1px solid #eee5dd;
  white-space: nowrap;
  overflow: hidden;
  box-sizing: border-box;
  flex: none;
}

.content-members .info-label {
  min-width: 48px;
  margin-right: 0.5rem;
  padding: 0;
}

.content-members .info-content {
  display: flex;
  align-items: center;
  gap: 0.3rem;
  padding-left: 0;
}

.number-input {
  width: 48px;
  height: 24px;
  padding: 0;
  border: none;
  border-radius: 20px;
  font-size: 0.9rem;
  color: #4b3621;
  background: transparent;
  text-align: center;
  font-weight: 600;
  box-sizing: border-box;
}

.edit-btn,
.join-btn,
.leave-btn,
.login-btn {
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  font-weight: 600;
  transition: all 0.2s ease;
  cursor: pointer;
}

.edit-btn {
  background-color: #eee5dd;
  color: #6f4e37;
  border: 1px solid #c4b5a5;
}

.edit-btn:hover {
  background-color: #e3d8ce;
}

.join-btn {
  background-color: #6f4e37;
  color: white;
  border: none;
}

.join-btn:hover {
  background-color: #5a3f2e;
}

.join-btn:disabled {
  background-color: #c4b5a5;
  cursor: not-allowed;
}

.leave-btn {
  background-color: #f5f2ef;
  color: #6f4e37;
  border: 1px solid #e3d8ce;
}

.leave-btn:hover {
  background-color: #eee5dd;
}

.login-btn {
  background-color: #6f4e37;
  color: white;
  border: none;
}

.login-btn:hover {
  background-color: #5a3f2e;
}

.delete-btn {
  padding: 0.75rem 1.5rem;
  background-color: #8b6b4a;
  color: white;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  transition: all 0.2s ease;
  cursor: pointer;
  margin-left: 0.5rem;
}

.delete-btn:hover {
  background-color: #6f4e37;
}

/* 스크롤바 관련 스타일 수정 */
.study-detail::-webkit-scrollbar,
.study-detail::-webkit-scrollbar-track,
.study-detail::-webkit-scrollbar-thumb,
.study-detail::-webkit-scrollbar-thumb:hover,
.right-section::-webkit-scrollbar,
.right-section::-webkit-scrollbar-track,
.right-section::-webkit-scrollbar-thumb,
.right-section::-webkit-scrollbar-thumb:hover {
  display: none !important;
}

.content-title {
  font-size: 1.8rem;
  font-weight: 700;
  color: #4b3621;
  flex: 1;
  max-width: 100%;
  width: 100%;
  display: block;
  padding-bottom: 0;
  line-height: 1.4;
  min-height: 2.52rem; /* 1.8rem * 1.4 */
}

.title-input {
  font-size: 1.8rem;
  font-weight: 700;
  color: #4b3621;
  padding: 0;
  border: none;
  border-bottom: 2px solid #eee5dd;
  background-color: transparent;
  width: 100%;
  transition: all 0.2s ease;
  line-height: 1.4;
  height: 2.52rem; /* 1.8rem * 1.4 */
  box-sizing: border-box;
}

.title-input:hover {
  border-color: #c4b5a5;
  background-color: #fdfbf9;
}

.title-input:focus {
  outline: none;
  border-color: #6f4e37;
  background-color: #fff;
}

.form-group::-webkit-scrollbar,
.form-group::-webkit-scrollbar-track,
.form-group::-webkit-scrollbar-thumb,
.form-group::-webkit-scrollbar-thumb:hover {
  display: none !important;
}

.form-group + .form-group {
  margin-top: 0;
  margin-bottom: 0.25rem;
}

.content-text {
  font-size: 1.1rem;
  line-height: 1.6;
  color: #4b3621;
  white-space: pre-line;
  margin-bottom: 0.25rem;
  padding: 1rem;
  background-color: #fbf9f8;
  border-radius: 8px;
  border: 1px solid #eee5dd;
  height: 320px;
  overflow-y: auto;
  flex-shrink: 0;
}

.content-location,
.content-date {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  font-size: 1rem;
  color: #6f4e37;
  margin-bottom: 0;
  margin-top: 0;
  padding: 0.5rem 0;
  width: 100%;
  max-width: 800px;
}

.info-label {
  min-width: 60px;
  font-weight: 600;
  color: #8b6b4a;
  flex-shrink: 0;
  text-align: right;
}

.info-content {
  display: flex;
  align-items: center;
  gap: 0.25rem;
  flex: 1;
  min-width: 0;
  padding-left: 2.25rem;
  position: relative;
}

.info-content i {
  color: #8b6b4a;
  font-size: 1rem;
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
  z-index: 1;
}

.location-dropdowns {
  display: flex;
  gap: 0.75rem;
  margin-bottom: 0;
  flex: 1;
  max-width: 600px;
}

.location-dropdowns .form-select {
  background-color: #f5f2ef;
  border: 1.5px solid #e3d8ce;
  transition: border-color 0.2s, box-shadow 0.2s, background-color 0.2s;
}

.location-dropdowns .form-select:focus {
  background-color: #fffbe9;
  border-color: #6f4e37;
  box-shadow: 0 0 0 2px #ffe6b8;
  outline: none;
  z-index: 2;
}

.location-dropdowns .form-select:hover {
  background-color: #fdf6e9;
  border-color: #c4b5a5;
}

.date-inputs {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  flex: 1;
  min-width: 0;
  max-width: 600px;
  margin: 0;
  padding: 0;
}

.date-picker-wrapper {
  position: relative;
  flex: 1;
  min-width: 0;
  height: 36px;
  cursor: pointer;
  margin: 0;
  padding: 0;
}

.date-input {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0;
  cursor: pointer;
  z-index: 3;
}

.date-display {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  padding: 0.4rem 0.75rem 0.4rem 2.25rem;
  border: 1px solid #e3d8ce;
  border-radius: 6px;
  font-size: 0.9rem;
  color: #4b3621;
  background-color: #fff;
  transition: all 0.2s ease;
  line-height: 28px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  cursor: pointer;
  z-index: 2;
  display: flex;
  align-items: center;
  pointer-events: none;
}

.date-display::before {
  content: '';
  position: absolute;
  left: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  width: 16px;
  height: 16px;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%236f4e37' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Crect x='3' y='4' width='18' height='18' rx='2' ry='2'%3E%3C/rect%3E%3Cline x1='16' y1='2' x2='16' y2='6'%3E%3C/line%3E%3Cline x1='8' y1='2' x2='8' y2='6'%3E%3C/line%3E%3Cline x1='3' y1='10' x2='21' y2='10'%3E%3C/line%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: center;
  background-size: contain;
  pointer-events: none;
  z-index: 1;
}

.date-picker-wrapper:hover .date-display {
  border-color: #c4b5a5;
  background-color: #fdfbf9;
}

.date-picker-wrapper:focus-within .date-display {
  border-color: #6f4e37;
  box-shadow: 0 0 0 2px rgba(111, 78, 55, 0.1);
}

.date-separator {
  color: #6f4e37;
  font-weight: 500;
  margin: 0 0.5rem;
  flex-shrink: 0;
  display: flex;
  align-items: center;
}

.kick-btn {
  display: none;
  margin-left: auto;
  background: #eee5dd;
  color: #8b6b4a;
  border: 1px solid #e3d8ce;
  border-radius: 12px;
  min-width: 40px;
  height: 28px;
  font-size: 0.85rem;
  font-weight: 400;
  cursor: pointer;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
  padding: 0 1rem;
}

.participants-list .participant-item:hover .kick-btn {
  display: inline-flex;
}

.kick-btn:hover {
  background: #e3d8ce;
  color: #4b3621;
}
</style> 