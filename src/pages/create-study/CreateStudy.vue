<template>
  <div class="create-study-container">
    <h1 class="page-title">스터디 만들기</h1>
    <div class="study-detail">
      <div class="left-section">
        <div class="thumbnail-section">
          <div class="thumbnail-preview" @click="triggerFileInput">
            <div v-if="thumbnailPreview" class="thumbnail-wrapper">
              <img :src="thumbnailPreview" alt="썸네일 미리보기" @error="handleImageError">
              <button class="delete-thumbnail" @click.stop="deleteThumbnail">
                <i class="fas fa-times"></i>
              </button>
            </div>
            <span v-else>사진 등록</span>
            <input 
              type="file" 
              id="thumbnail" 
              ref="fileInput"
              @change="handleThumbnailChange" 
              accept="image/*"
              required
              style="display: none"
            >
          </div>
        </div>
      </div>
      <div class="right-section">
        <form @submit.prevent="handleFormSubmit" class="text-start">
          <div class="form-group title-category-group">
            <div class="content-title">
              <input 
                type="text" 
                id="title" 
                v-model="formData.title" 
                required 
                placeholder="스터디 제목을 입력하세요"
                class="title-input"
              >
            </div>
            <div class="category-members-group">
              <div class="content-category">
                <select 
                  id="category" 
                  v-model="formData.categoryId" 
                  required
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
                  <input 
                    type="number" 
                    id="maxMembers" 
                    v-model="formData.maxMembers" 
                    required 
                    min="1" 
                    max="999"
                    class="number-input"
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
                <div class="location-dropdowns">
                  <select v-model="selectedSido" @change="handleSidoChange" class="form-select" required>
                    <option value="">시/도 선택</option>
                    <option v-for="sido in sidoList" :key="sido.id" :value="sido.id">{{ sido.name }}</option>
                  </select>
                  <select v-model="selectedSigungu" @change="handleSigunguChange" class="form-select" :disabled="!selectedSido" required>
                    <option value="">시/군/구 선택</option>
                    <option v-for="sigungu in sigunguList" :key="sigungu.id" :value="sigungu.id">{{ sigungu.name }}</option>
                  </select>
                  <select v-model="selectedDong" class="form-select" :disabled="!selectedSigungu" required>
                    <option value="">읍/면/동 선택</option>
                    <option v-for="dong in dongList" :key="dong.id" :value="dong.id">{{ dong.name }}</option>
                  </select>
                </div>
              </span>
            </div>
          </div>

          <div class="form-row">
            <div class="form-group">
              <div class="content-date">
                <span class="info-label">기간</span>
                <span class="info-content">
                  <i class="fas fa-calendar-alt"></i>
                  <div class="date-inputs">
                    <div class="date-picker-wrapper" @click="focusDateInput('startDate')">
                      <input 
                        type="date" 
                        id="startDate" 
                        v-model="formData.startDate" 
                        required
                        class="date-input"
                        :min="getTodayDate()"
                        @change="validateDates"
                      >
                      <div class="date-display">
                        {{ formatDate(formData.startDate) }}
                      </div>
                    </div>
                    <span class="date-separator">~</span>
                    <div class="date-picker-wrapper" @click="focusDateInput('endDate')">
                      <input 
                        type="date" 
                        id="endDate" 
                        v-model="formData.endDate" 
                        required
                        class="date-input"
                        :min="formData.startDate || getTodayDate()"
                        @change="validateDates"
                      >
                      <div class="date-display">
                        {{ formatDate(formData.endDate) }}
                      </div>
                    </div>
                  </div>
                </span>
              </div>
            </div>
          </div>

          <div class="form-group">
            <div class="content-text">
              <textarea 
                id="content" 
                v-model="formData.content" 
                required 
                placeholder="스터디에 대한 설명을 입력하세요"
                class="form-textarea"
              ></textarea>
            </div>
          </div>

          <div class="form-actions">
            <button type="button" class="cancel-btn" @click="handleCancel">취소</button>
            <button type="submit" class="submit-btn">스터디 만들기</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useCreateStudy } from './scripts/useCreateStudy'
import axios from 'axios'

const router = useRouter()
const { formData, handleSubmit, handleThumbnailChange, thumbnailPreview } = useCreateStudy()

// formData 초기화
formData.value = {
  ...formData.value,
  categoryId: ''
}

// 카테고리 데이터
const categories = ref([])

// 카테고리 데이터 가져오기
const fetchCategories = async () => {
  try {
    const res = await axios.get('http://localhost:3000/category')
    categories.value = res.data.data
  } catch (error) {
    console.error('카테고리 로딩 실패:', error)
  }
}

// 지역 선택 관련 상태
const selectedSido = ref('')
const selectedSigungu = ref('')
const selectedDong = ref('')
const selectedLocation = ref('')
const sidoList = ref([])
const sigunguList = ref([])
const dongList = ref([])

// 파일 입력 트리거 함수 추가
const fileInput = ref(null)

const triggerFileInput = () => {
  fileInput.value.click()
}

// 시/도 목록 가져오기
const fetchSidoList = async () => {
  try {
    const res = await axios.get('http://localhost:3000/city')
    sidoList.value = res.data.data
  } catch (e) {
    console.error('시/도 목록 불러오기 실패', e)
  }
}

// 시/군/구 목록 가져오기
const fetchSigunguList = async (cityId) => {
  try {
    const res = await axios.get(`http://localhost:3000/district/${cityId}`)
    sigunguList.value = res.data.data
  } catch (e) {
    console.error('시/군/구 목록 불러오기 실패', e)
  }
}

// 읍/면/동 목록 가져오기
const fetchDongList = async (districtId) => {
  try {
    const res = await axios.get(`http://localhost:3000/town/${districtId}`)
    dongList.value = res.data.data
  } catch (e) {
    console.error('읍/면/동 목록 불러오기 실패', e)
  }
}

// 시/도 변경 핸들러
const handleSidoChange = async () => {
  selectedSigungu.value = ''
  selectedDong.value = ''
  selectedLocation.value = ''
  sigunguList.value = []
  dongList.value = []
  
  if (selectedSido.value) {
    await fetchSigunguList(selectedSido.value)
  }
}

// 시/군/구 변경 핸들러
const handleSigunguChange = async () => {
  selectedDong.value = ''
  selectedLocation.value = ''
  dongList.value = []
  
  if (selectedSigungu.value) {
    await fetchDongList(selectedSigungu.value)
  }
}

// 컴포넌트 마운트 시 시/도 목록과 카테고리 목록 가져오기
onMounted(async () => {
  await Promise.all([
    fetchSidoList(),
    fetchCategories()
  ])
})

// 폼 제출 시 지역 정보 추가
const handleFormSubmit = async () => {
  if (!selectedLocation.value) {
    alert('지역을 선택해주세요.')
    return
  }
  
  formData.value.location = {
    sido: selectedSido.value,
    sigungu: selectedSigungu.value,
    dong: selectedDong.value
  }
  await handleSubmit()
}

// 취소 버튼 클릭 처리
const handleCancel = () => {
  // 저장된 카테고리 정보 가져오기
  const savedCategory = localStorage.getItem('lastSelectedCategory')
  if (savedCategory) {
    const category = JSON.parse(savedCategory)
    router.push({
      path: '/',
      query: { 
        category: category.id,
        categoryName: category.name 
      }
    })
  } else {
    router.push('/')
  }
}

// 이미지 로드 에러 처리
const handleImageError = (e) => {
  e.target.src = '' // 이미지 로드 실패 시 빈 이미지로 설정
  thumbnailPreview.value = '' // 미리보기 초기화
}

// 썸네일 삭제 함수 추가
const deleteThumbnail = () => {
  thumbnailPreview.value = ''
  formData.value.thumbnail = null
  if (fileInput.value) {
    fileInput.value.value = ''
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

// 날짜 유효성 검사
const validateDates = () => {
  if (formData.value.startDate && formData.value.endDate) {
    const startDate = new Date(formData.value.startDate)
    const endDate = new Date(formData.value.endDate)
    
    if (endDate < startDate) {
      alert('종료일은 시작일보다 이후여야 합니다.')
      formData.value.endDate = formData.value.startDate
    }
  }
}

// 날짜 포맷팅 함수
const formatDate = (dateString) => {
  if (!dateString) return '날짜 선택'
  const date = new Date(dateString)
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}년 ${month}월 ${day}일`
}

// 날짜 입력 필드 포커스 함수 추가
const focusDateInput = (inputId) => {
  const input = document.getElementById(inputId)
  if (input) {
    input.showPicker()
  }
}
</script>

<style scoped>
.create-study-container {
  background-color: #faf7f5;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  box-sizing: border-box;
  padding-left: 2rem;
}

.study-detail {
  display: flex;
  gap: 2rem;
  background-color: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 1300px;
  min-height: 570px;
  box-sizing: border-box;
  margin-top: 0;
}

.left-section {
  width: 300px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.thumbnail-section {
  width: 100%;
  aspect-ratio: 4/4;
  overflow: hidden;
  border-radius: 8px;
  border: 1px solid #eee5dd;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
}

.thumbnail-preview {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  border-radius: 8px;
  color: #8b6b4a;
  font-size: 0.9rem;
  text-align: center;
  padding: 1rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

.thumbnail-preview:hover {
  background-color: #faf7f5;
}

.thumbnail-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
}

.delete-thumbnail {
  position: absolute;
  top: -8px;
  right: -8px;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background-color: #eee5dd;
  border: 1px solid #e3d8ce;
  color: #6f4e37;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s ease;
  padding: 0;
  font-size: 0.7rem;
  font-weight: 500;
  line-height: 1;
  z-index: 1;
}

.delete-thumbnail:hover {
  background-color: #e3d8ce;
  color: #4b3621;
}

.delete-thumbnail i {
  display: none;
}

.delete-thumbnail::after {
  content: "×";
  font-size: 1rem;
  line-height: 1;
  margin-top: -1px;
}

.thumbnail-preview img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.right-section {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
}

.page-title {
  color: #4b3621;
  font-size: 1.5rem;
  font-weight: 700;
  margin-bottom: 1rem;
  margin-top: 1.25rem;
  text-align: left;
  width: 100%;
  margin-left: 0rem;
  position: relative;
  left: 6rem;
}

.title-category-group {
  display: flex;
  align-items: center;
  gap: 1.2rem;
  margin-bottom: 1.2rem;
  width: 100%;
  border-bottom: 1px solid #eee5dd;
  padding-bottom: 1rem;
}

.content-title {
  flex: 2 1 0;
  max-width: 50%;
  min-width: 200px;
  margin-right: 1.2rem;
  display: flex;
  align-items: center;
}

.title-input {
  font-size: 1.8rem;
  font-weight: 700;
  color: #4b3621;
  padding: 0.5rem 0.75rem;
  border: none;
  background: transparent;
  width: 100%;
  min-width: 120px;
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

.content-category,
.content-members {
  display: inline-block;
  min-width: 120px;
  height: 36px;
  padding: 0.5rem 0.7rem;
  font-size: 0.87rem;
  box-sizing: border-box;
  margin: 0;
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
  height: 36px;
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
}

.content-members {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  min-width: 120px;
  font-size: 0.87rem;
  color: #6f4e37;
  padding: 0.5rem 0.3rem 0.5rem 0.3rem;
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
  margin-right: 0.2rem;
  padding: 0;
}

.content-members .info-content {
  display: flex;
  align-items: center;
  gap: 0.1rem;
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

.content-location {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  font-size: 1rem;
  color: #6f4e37;
  margin-bottom: 0.3rem;
  margin-top: 0.5rem;
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
  margin-bottom: 0.5rem;
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

.content-date {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 1rem;
  color: #6f4e37;
  margin-bottom: 0;
  margin-top: 0;
  padding: 0.5rem 0;
  width: 100%;
  max-width: 800px;
}

.content-date .info-label {
  min-width: 60px;
  font-weight: 600;
  color: #8b6b4a;
  flex-shrink: 0;
  text-align: right;
  margin-bottom: 0;
}

.content-date .info-content {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  flex: 1;
  min-width: 0;
  max-width: 600px;
  position: relative;
  padding-left: 2.25rem;
}

.content-date .info-content i {
  color: #8b6b4a;
  font-size: 1rem;
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
  z-index: 1;
}

.date-inputs {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  flex: 1;
  min-width: 0;
  max-width: 600px;
}

.date-picker-wrapper {
  position: relative;
  flex: 1;
  min-width: 0;
  height: 36px;
  cursor: pointer;
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

.content-text {
  font-size: 1.1rem;
  line-height: 1.6;
  color: #4b3621;
  white-space: pre-line;
  margin-bottom: 0.7rem;
  padding: 1rem;
  background-color: #fbf9f8;
  border-radius: 8px;
  height: 335px;
  flex-shrink: 0;
}

.form-textarea {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #e3d8ce;
  border-radius: 8px;
  font-size: 1.1rem;
  line-height: 1.6;
  color: #4b3621;
  background-color: #fbf9f8;
  transition: all 0.2s ease;
  height: 300px;
  resize: none;
  overflow-y: auto;
  margin-bottom: 0.25rem;
}

.form-textarea:hover {
  border-color: #c4b5a5;
  background-color: #fdfbf9;
}

.form-textarea:focus {
  outline: none;
  border-color: #6f4e37;
  box-shadow: 0 0 0 2px rgba(111, 78, 55, 0.1);
  background-color: #fff;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 1rem;
  margin-top: auto;
  padding-top: 1rem;
  border-top: 1px solid #e3d8ce;
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

@media (max-width: 900px) {
  .title-category-group {
    flex-direction: column;
    align-items: stretch;
    gap: 1rem;
  }
  .content-title {
    max-width: 100%;
    margin-right: 0;
  }
  .category-members-group {
    flex-direction: column;
    align-items: stretch;
    gap: 0.7rem;
    justify-content: flex-start;
    margin-left: 0;
  }
}

@media (max-width: 768px) {
  .create-study-container {
    padding-left: 1rem;
  }
  .page-title {
    font-size: 1.1rem;
    margin-top: 0.5rem;
    margin-bottom: 0.5rem;
  }
}
</style> 