<script setup>
import { onMounted, ref, computed, watch } from 'vue';
import { useMovieStore } from '../stores/movieStore';

const store = useMovieStore();

// 추가 미션 [선택 1] - 정렬 기능
const sortKey = ref('');
const sortDirection = ref('asc');

const setSort = (key) => {
  if (sortKey.value === key) {
    sortDirection.value = sortDirection.value === 'asc' ? 'desc' : 'asc';
  } else {
    sortKey.value = key;
    if (key === 'title') sortDirection.value = 'asc';
    else sortDirection.value = 'desc';
  }
};

const displayedMovies = computed(() => {
  const arr = [...filteredMovies.value];
  if (!sortKey.value) return arr;

  if (sortKey.value === 'title') {
    return arr.sort((a, b) => a.title.localeCompare(b.title, 'ko') * (sortDirection.value === 'asc' ? 1 : -1));
  }

  if (sortKey.value === 'date') {
    return arr.sort((a, b) => {
      const ta = a.release_date ? new Date(a.release_date).getTime() : 0;
      const tb = b.release_date ? new Date(b.release_date).getTime() : 0;
      return (ta - tb) * (sortDirection.value === 'asc' ? 1 : -1);
    });
  }

  if (sortKey.value === 'rating') {
    return arr.sort((a, b) => (a.vote_average - b.vote_average) * (sortDirection.value === 'asc' ? 1 : -1));
  }

  return arr;
});

// 추가 미션 [선택 2] - 검색 기능
const searchQuery = ref('');

const filteredMovies = computed(() => {
  const q = searchQuery.value.trim().toLowerCase();
  if (!q) return [...store.movies];

  return store.movies.filter(m => {
    const title = (m.title || '').toLowerCase();
    const overview = (m.overview || '').toLowerCase();
    const original = (m.original_title || '').toLowerCase();
    return title.includes(q) || overview.includes(q) || original.includes(q);
  });
});

// 추가 미션 [선택 4] - 페이지네이션 기능
const currentPage = ref(1);
const pageSize = ref(12);
const totalPages = computed(() => Math.max(1, Math.ceil(displayedMovies.value.length / pageSize.value)));
const pagesArray = computed(() => Array.from({ length: totalPages.value }, (_, i) => i + 1));

// 페이징된 데이터
const pagedMovies = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value;
  return displayedMovies.value.slice(start, start + pageSize.value);
});

// 페이지 변경 시 유효성 체크 및 스크롤
const setPage = (n) => {
  const num = Number(n);
  if (Number.isNaN(num)) return;
  if (num < 1) currentPage.value = 1;
  else if (num > totalPages.value) currentPage.value = totalPages.value;
  else currentPage.value = num;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};
const prevPage = () => setPage(currentPage.value - 1);
const nextPage = () => setPage(currentPage.value + 1);

// 검색어나 정렬이 변경되면 페이지를 1로 초기화
watch([searchQuery, sortKey, sortDirection, displayedMovies], () => {
  if (currentPage.value > totalPages.value) currentPage.value = totalPages.value;
  if (currentPage.value === 0) currentPage.value = 1;
});
onMounted(() => {
     if (store.movies.length === 0) {
        store.fetchMovies();
    }
    document.title = '🍿 국내 극장 화제작 (인기순)';
});
</script>

<template>
    <main class="page">
        <div class="header-section">
            <h1>🍿 국내 극장 화제작 (인기순)</h1>
            <p class="sub-title">2025년 이후 국내 정식 개봉한 실시간 인기 상영작</p>
            <div class="search-box">
              <input
                v-model="searchQuery"
                class="search-input"
                type="search"
                placeholder="제목 또는 줄거리 입력 후 엔터로 검색"
                aria-label="영화 검색"
              />
              <div v-if="searchQuery && displayedMovies.length === 0" class="no-results">🔍 검색 결과가 없습니다.</div>
            </div>

            <div class="sort-controls">
                <button @click="setSort('title')" :class="{ active: sortKey === 'title' }">제목순</button>
                <button @click="setSort('date')" :class="{ active: sortKey === 'date' }">개봉일순</button>
                <button @click="setSort('rating')" :class="{ active: sortKey === 'rating' }">평점순</button>
            </div>
        </div>
        <div v-if="store.isLoading" class="status-message loading">
            ⏳ 실시간 국내 개봉작 데이터를 싣고 오는 중입니다...
        </div>
        <div v-else-if="store.errorMessage" class="status-message error">
            🚨 {{ store.errorMessage }}
        </div>
        <div v-else>
            <div class="movie-list">
                <div v-for="movie in pagedMovies" :key="movie.id" class="movie-card">
                    <img
                        v-if="movie.poster_path"
                        :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
                        :alt="movie.title"
                        class="poster"
                    />
                    <div v-else class="poster-placeholder">이미지 준비 중</div>
                    <div class="card-content">
                        <h3 class="title">{{ movie.title }}</h3>
                        <p class="release-date" v-if="movie.release_date">📅 개봉일: {{ movie.release_date }}</p>
                        <p class="rating">⭐ {{ movie.vote_average.toFixed(1) }} / 10</p>
                        <p class="overview">
                            {{ movie.overview ? movie.overview.substring(0, 60) + '...' : '국내에 등록된 줄거리 요약 정보가 없습니다.' }}
                        </p>
                        <button
                            @click="store.toggleFavorite(movie.id)"
                            :class="{ active: movie.isFavorite }"
                            class="fav-btn"
                        >
                            {{ movie.isFavorite ? '❤️ 찜 해제' : '🤍 찜하기' }}
                        </button>
                    </div>
                    <RouterLink
                        :to="`/movies/${movie.id}`"
                        class="stretched-link"
                        :aria-label="`${movie.title} 상세 정보 보기`"
                    ></RouterLink>
                </div>
            </div>
            
            <nav class="pagination" v-if="totalPages > 1">
                <button
                v-for="p in pagesArray"
                :key="p"
                class="page-number"
                :class="{ active: p === currentPage }
                "
                @click="setPage(p)"
                >
                {{ p }}
                </button>
            </nav>
        </div>
    </main>
</template>

<style scoped>
.page { padding: 40px; background-color: #f8f9fa; min-height: 100vh; }
.header-section { margin-bottom: 30px; text-align: center; color: #2c3e50; }
.sub-title { font-size: 14px; color: #7f8c8d; margin-top: 5px; }

.search-box { display:flex; justify-content:center; gap:12px; align-items:center; margin-top:12px; }
.search-input { width: 420px; max-width: 80%; padding: 10px 14px; border-radius: 8px; border: 1px solid #dcdfe6; font-size: 14px; }
.no-results { color: #7f8c8d; font-size: 13px; }

.sort-controls { display:flex; gap:10px; justify-content:center; margin-top:12px; }
.sort-controls button { padding:8px 12px; border-radius:8px; border:none; background:#ecf0f1; cursor:pointer; font-weight:700; }
.sort-controls button.active { background:#ff4757; color:white; }
.sort-controls button small { margin-left:6px; font-weight:600; font-size:12px; color:rgba(0,0,0,0.6); }

.status-message { text-align: center; font-size: 20px; font-weight: bold; padding: 50px; border-radius: 12px; }
.loading { color: #3498db; background-color: #e3f2fd; }
.error { color: #e74c3c; background-color: #fdeaea; }

.movie-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 30px; }

.movie-card { 
    position: relative;
    border-radius: 12px; 
    overflow: hidden; 
    background: white; 
    text-align: left; 
    box-shadow: 0 4px 15px rgba(0,0,0,0.05); 
    transition: transform 0.2s ease; 
    display: flex; 
    flex-direction: column; 
}
.movie-card:hover { transform: translateY(-5px); }

.poster { width: 100%; height: 380px; object-fit: cover; }
.poster-placeholder { 
    width: 100%; 
    height: 380px; 
    background-color: #ddd; 
    display: flex; 
    align-items: center; 
    justify-content: center; 
    color: #7f8c8d; 
    font-weight: bold; 
}

.card-content { padding: 20px; display: flex; flex-direction: column; flex-grow: 1; }
.title { 
    font-size: 18px; 
    color: #333; 
    margin: 0 0 6px 0; 
    white-space: nowrap; 
    overflow: hidden; 
    text-overflow: ellipsis; 
    font-weight: bold; 
}
.release-date { font-size: 13px; color: #7f8c8d; margin-bottom: 10px; font-weight: 500; }
.rating { font-weight: bold; color: #f39c12; margin-bottom: 10px; font-size: 16px; }
.overview { font-size: 13px; color: #555; line-height: 1.4; margin-bottom: 20px; flex-grow: 1; }

.fav-btn { 
    position: relative;
    z-index: 2;
    width: 100%; 
    padding: 12px; 
    cursor: pointer; 
    border: none; 
    background: #ecf0f1; 
    color: #333; 
    border-radius: 8px; 
    font-weight: bold; 
    font-size: 14px; 
    transition: 0.3s; 
    margin-top: auto; 
}
.fav-btn.active { background: #ff4757; color: white; }
.stretched-link {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 1;
}
.pagination { display:flex; gap:8px; justify-content:center; align-items:center; margin: 28px 0 10px; }
.page-number, .page-btn { padding:8px 12px; border-radius:8px; border:1px solid #e6e9ef; background:#fff; cursor:pointer; font-weight:700; }
.page-number.active { background:#ff4757; color:#fff; border-color:#ff4757; }

</style>