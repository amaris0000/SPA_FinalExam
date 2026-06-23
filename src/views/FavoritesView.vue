<script setup>
// 추가 미션 [선택 3] - 찜 목록 조회 기능
import { computed } from 'vue';
import { RouterLink } from 'vue-router';
import { useMovieStore } from '../stores/movieStore';

const store = useMovieStore();

const favorites = computed(() => store.favorites);

const toggleFav = (id) => store.toggleFavorite(id);
</script>

<template>
  <main class="page">
    <div class="header-section">
      <h1>❤️ 찜한 작품</h1>
      <p class="sub-title">사용자가 찜한 작품 목록을 보여줍니다.</p>
    </div>

    <div v-if="favorites.length === 0" class="status-message empty">
      아직 찜한 작품이 없습니다. 영화 목록에서 마음에 드는 작품을 찜해보세요.
    </div>

    <div v-else class="movie-list">
      <div v-for="movie in favorites" :key="movie.id" class="movie-card">
        <img
          v-if="movie.poster_path"
          :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
          :alt="movie.title"
          class="poster"
        />
        <div v-else class="poster-placeholder">이미지 준비 중</div>

        <div class="card-content">
          <h3 class="title">{{ movie.title }}</h3>
          <p class="description">상세 정보를 보려면 클릭하세요.</p>

          <button
            @click="toggleFav(movie.id)"
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
  </main>
</template>

<style scoped>
.page { padding: 40px; background-color: #f8f9fa; min-height: 100vh; }
.header-section { margin-bottom: 30px; text-align: center; color: #2c3e50; }
.sub-title { font-size: 14px; color: #7f8c8d; margin-top: 5px; }

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
.description { 
    font-size: 14px; 
    color: #7f8c8d; 
    margin: 0 0 12px 0; 
    flex-grow: 1; 
}

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
.status-message.empty {
  text-align: center;
  font-size: 16px;
  color: #7f8c8d;
  margin-top: 50px;
}
</style>
