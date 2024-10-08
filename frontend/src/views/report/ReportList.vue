<template>
  <div class="report-list-section">
    <h2>신고 목록</h2>
    <div class="report-list">
      <div class="report-header">
        <span>개수 : {{ filteredItems.length }}개</span>
        <input type="text" v-model="searchQuery" placeholder="ID로 검색하세요" />
      </div>
      <ul>
        <li v-for="item in paginatedItems" :key="item.id" class="report-item" @click="goToReportRead(item)">
          <!-- <span class="report-type">{{ item.type }}</span> -->
          <span class="report-reason">{{ item.reason }}</span>
          <span class="report-date">{{ formatDate(item.createdAt) }}</span>
        </li>
      </ul>
      <div class="pagination">
        <button @click="prevPage" :disabled="currentPage === 1">&lt;</button>
        <span>{{ currentPage }} / {{ totalPages }}</span>
        <button @click="nextPage" :disabled="currentPage === totalPages">&gt;</button>
      </div>
      <!-- <button @click="goToCreatereport" class="create-btn">등록</button> -->
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

const reportItems = ref([])
const currentPage = ref(1)
const itemsPerPage = 10
const searchQuery = ref('')

// 검색어에 따른 필터링된 리스트
const filteredItems = computed(() => {
  if (!searchQuery.value) {
    return reportItems.value // 검색어가 없을 경우 전체 리스트 반환
  }
  return reportItems.value.filter(item => (item.reporterId === parseInt(searchQuery.value)||(item.respondentId === parseInt(searchQuery.value))))
})

const formatDate = (dateString) => {
    if (!dateString) return 'N/A'
    const options = { year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit' }
    return new Date(dateString).toLocaleDateString('ko-KR', options)
  }

const totalItems = computed(() => filteredItems.value.length)
const totalPages = computed(() => Math.ceil(totalItems.value / itemsPerPage))

// 현재 페이지에 맞게 필터링된 리스트에서 페이징 처리된 결과 반환
const paginatedItems = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  const end = start + itemsPerPage
  return filteredItems.value.slice(start, end)
})

// 데이터를 불러오는 예시 함수
const fetchreportItems = async () => {
  // reportItems.value = [
  //   { id: 1, type: '필독', title: '펫케어몰레터의 테마 선택 기준 여부', date: '24.09.01', reporterId: 3, respondentId: 1},
  //   { id: 2, type: '필독', title: '펫베스트와 행사 기획 범위', date: '24.08.13', reporterId: 5, respondentId: 4 },
  //   { id: 3, type: '일반', title: '도그웨딩브레이션 케이터링 서비스', date: '24.08.05', reporterId: 1, respondentId: 2 },
  //   // 더 많은 데이터 추가 가능
  // ]
  try{
    const response = await fetch('http://localhost:8888/reports');
    if(!response.ok){
      throw new Error('이상해여');
    }
    const data = await response.json();
    // console.log(data);
    reportItems.value = data;

  }catch(error){
    console.error(error);
  }

}

const prevPage = () => {
  if (currentPage.value > 1) currentPage.value--
}

const nextPage = () => {
  if (currentPage.value < totalPages.value) currentPage.value++
}

// const goToCreatereport = () => {
//   router.push('/api/v1/report/post') // report 작성 페이지로 이동
// }

const goToReportRead = (item) => {
  router.push({path: '/api/v1/report/${item.id}',
    query: {
      id: item.id,
      reason: item.reason,
      createdAt: item.createdAt,
      reporterId: item.reporterId,
      respondentId: item.respondentId
    }
  }
  ) // report 상세 읽기 페이지로 이동
}

onMounted(() => {
  fetchreportItems()
})
</script>

<style scoped>
.report-list-section {
  font-family: 'Arial', sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
}

h2 {
  color: #333;
  margin-bottom: 20px;
  font-size: 24px;
  font-weight: bold;
  /* font-family: Jua; */
  display: flex;
  justify-content:flex-start
}

.report-list {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.report-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.report-header input {
  padding: 5px;
  border: 1px solid #ddd;
  border-radius: 4px;
  width: 250px;
}

ul {
  list-style-type: none;
  padding: 0;
}

.report-item {
  display: flex;
  justify-content: space-between;
  padding: 15px;
  margin-bottom: 10px;
  background-color: #DDF5FF;  /* 목록 항목의 배경색 */
  border: 1px solid #B0DFF7;  /* 테두리 색상 */
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.report-item:hover {
  background-color: #C7E9FF; /* 호버 시 더 진한 파란색 */
}

.report-type {
  background-color: #f6e7e7; /* 유형의 배경색 */
  padding: 5px 12px;
  border-radius: 12px;
  font-size: 0.8em;
  color: #000000;  /* 텍스트 색상 */
}

.report-title {
  flex-grow: 1;
  margin: 0 15px;
  font-weight: bold;
  color: #333;  /* 제목 텍스트 색상 */
}

.report-date {
  color: #888;
  font-size: 0.9em;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

.pagination button {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 10px 20px;
  margin: 0 5px;
  cursor: pointer;
  border-radius: 4px;
  font-size: 14px;
}

.pagination button:disabled {
  background-color: #ddd;
  cursor: not-allowed;
}

.create-btn {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 10px 20px;
  margin-top: 50px;
  cursor: pointer;
  border-radius: 4px;
  float: right;
  display: flex;
  justify-content: center;
}

</style>
