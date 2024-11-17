<template>
  <div class="album-container">
    <div class="header">
      <div>
        <div class="logo">
        <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="30" height="30" viewBox="0 0 48 48">
          <path fill="#212121" d="M24.001,44.001c11.045,0,20-8.955,20-20s-8.955-20-20-20	c-11.045,0-20,8.955-20,20S12.956,44.001,24.001,44.001z"></path><path fill="#fcbe2d" d="M39.2,20.019l-0.129-0.607l-5.097-0.892l2.968-4.021	L36.6,14.104l-4.364,2.104l0.552-5.573l-0.447-0.261l-2.655,4.52l-2.971-6.728h-0.524l0.709,6.491l-7.492-6.019l-0.631,0.184	l5.757,7.281l-11.407-3.812l-0.527,0.58L22.8,18.705L8.739,19.887l-0.157,0.868l14.612,1.601L10.999,32.504l0.527,0.708	l14.508-7.937l-2.864,13.984h0.868l5.569-13.168L33,36.392l0.603-0.473L32.212,25.46l5.28,6.019l0.341-0.555l-4.045-7.463	l5.649,2.103l0.053-0.631l-5.072-3.76L39.2,20.019z"></path>
        </svg>
        <b class="name">Music Port</b>
        </div>
      </div>
      <div class="headerLeft">
      <div class="search-container">
        <input v-model="searchQuery" type="text" placeholder="Search" />
      </div>
      <div class="auth-buttons">
        <button @click="showModal('login')">Login</button>
        <button @click="showModal('register')">Register</button>
      </div>
      </div>
    </div>

    <h1 class="maintext" @click="refreshPage">Main page</h1>

    <div v-if="paginatedAlbums && paginatedAlbums.length">
      <div v-for="album in paginatedAlbums" :key="album.name" class="album-card">
        <a :href="album.url" target="_blank" class="album-link">
          <img
              :src="album.image.find(img => img.size === 'extralarge')?.['#text']"
              alt="Album Cover"
              class="album-image"
          />
          <div class="album-info">
            <h2 :title="album.name">{{ album.name }}</h2>
            <p>{{ album.artist }}</p>
          </div>
        </a>
      </div>
    </div>

    <div v-else>
      <p>Loading album data...</p>
    </div>
    <div v-if="isModalVisible" class="modal-overlay" @click="closeModal">
      <div class="modal" @click.stop>
        <h2>{{ modalType === 'login' ? 'Вход в систему' : 'Регистрация' }}</h2>
        <form>
          <input type="text" placeholder="Логин" />
          <input type="password" placeholder="Пароль" />
          <button type="submit">{{ modalType === 'login' ? 'Войти' : 'Зарегистрироваться' }}</button>
        </form>
        <button @click="closeModal" class="close-btn">Закрыть</button>
      </div>
    </div>

    <div class="pagination">
      <button
          :disabled="currentPage === 1"
          @click="changePage(currentPage - 1)">
        &lt;
      </button>

      <!-- Кнопки пагинации с номерами страниц -->
      <button
          v-for="pageNumber in displayedPages"
          :key="pageNumber"
          :class="{ active: pageNumber === currentPage }"
          @click="changePage(pageNumber)">
        {{ pageNumber }}
      </button>

      <button
          :disabled="currentPage === totalPages"
          @click="changePage(currentPage + 1)">
        &gt;
      </button>
    </div>
    <div> <h4>Alikhanova Zarina BDA-2302</h4></div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      apiData: null,
      searchQuery: '', // Для поиска
      currentPage: 1,  // Текущая страница
      albumsPerPage: 12,  // Количество альбомов на странице
      isModalVisible: false,  // Состояние видимости модального окна
      modalType: '',
    };
  },
  watch: {
    // Следим за изменениями поискового запроса
    searchQuery() {
      this.currentPage = 1; // При изменении поискового запроса сбрасываем на первую страницу
      this.fetchData();
    }
  },
  created() {
    this.fetchData(); // При создании компонента запрашиваем данные
  },
  methods: {
    showModal(type) {
      this.modalType = type;
      this.isModalVisible = true;
    },
    closeModal() {
      this.isModalVisible = false;
    },
    // Функция для запроса данных
    fetchData() {
      const apiKey = import.meta.env.VITE_LASTFM_API_KEY;
      axios.get(`https://ws.audioscrobbler.com/2.0/?method=album.search&album=Thriller&api_key=${apiKey}&format=json`)
          .then(response => {
            this.apiData = response.data;
          })
          .catch(error => {
            console.error(error);
          });
    },

    // Метод для обновления данных при клике на "Главное"
    refreshPage() {
      this.currentPage = 1; // Сброс страницы на 1 при обновлении данных
      this.fetchData(); // Запрашиваем новые данные
    },

    // Метод для изменения страницы
    changePage(pageNumber) {
      this.currentPage = pageNumber;
    },
  },
  computed: {
    // Фильтрация альбомов по запросу
    filteredAlbums() {
      if (!this.apiData || !this.apiData.results || !this.apiData.results.albummatches) return [];

      // Фильтрация по поисковому запросу
      const filtered = this.apiData.results.albummatches.album.filter(album => {
        const nameMatch = album.name.toLowerCase().includes(this.searchQuery.toLowerCase());
        const artistMatch = album.artist.toLowerCase().includes(this.searchQuery.toLowerCase());
        return nameMatch || artistMatch;
      });

      // Сортировка альбомов по имени
      return filtered.sort((a, b) => a.name.localeCompare(b.name));
    },

    // Пагинированные альбомы
    paginatedAlbums() {
      const startIndex = (this.currentPage - 1) * this.albumsPerPage;
      return this.filteredAlbums.slice(startIndex, startIndex + this.albumsPerPage);
    },

    // Общее количество страниц
    totalPages() {
      return Math.ceil(this.filteredAlbums.length / this.albumsPerPage);
    },

    // Отображаемые номера страниц для пагинации
    displayedPages() {
      const total = this.totalPages;
      let pages = [];

      if (total <= 9) {
        // Если страниц меньше или равно 9, отображаем все
        pages = Array.from({ length: total }, (_, i) => i + 1);
      } else {
        // В противном случае, выводим диапазон с тремя точками
        if (this.currentPage <= 5) {
          pages = [1, 2, 3, 4, 5, '...', total];
        } else if (this.currentPage >= total - 4) {
          pages = [1, '...', total - 4, total - 3, total - 2, total - 1, total];
        } else {
          pages = [1, '...', this.currentPage - 1, this.currentPage, this.currentPage + 1, '...', total];
        }
      }
      return pages;
    },
  },
};
</script>

<style scoped>


.auth-buttons button {
  background-color: #fcbe2d;
  border: none;
  padding: 8px 20px;
  color: #212121;
  font-size: 16px;
  cursor: pointer;
  margin: 0 5px 0 5px;
  border-radius: 4px;
}

.auth-buttons button:hover {
  background-color: #fcbf6b;
}
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(5px);
  z-index: 999;
}

.modal {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  width: 300px;
  text-align: center;
}

.modal input {
  width: 90%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.modal button {
  background-color: #fcbe2d;
  border: none;
  padding: 10px 20px;
  color: #212121;
  font-size: 16px;
  cursor: pointer;
  border-radius: 4px;
  width: 100%;
}

.modal button:hover {
  background-color: #fcbf6b;
}
.close-btn {
  background-color: #d9534f;
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
  margin-top: 10px;
}
.name {
  font-weight: bold;
  font-size: 26px;
}

.header {
  width: 100%;
  display: flex;
  justify-content: space-between;
  border-bottom: #464545 1px solid;
}

.logo {
  margin-top: 6px;
}

.album-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
  color: #ccc;
  background-color: #1a1a1a; /* Dark background */
  text-align: center;
  margin-top: -8px;
  height: 100%;
}

.maintext {
  margin-left: -770px;
  cursor: pointer; /* Указатель мыши меняется при наведении на "Главное" */
}

h1 {
  color: #fff;
  font-size: 2.5rem;
  margin-bottom: 30px;
}

.search-container {
  margin-bottom: 20px;
  margin-right: 66px;
}

.search-container input {
  padding: 10px;
  font-size: 1.2rem;
  width: 100%;
  max-width: 300px;
  margin: 0 auto;
  border: 2px solid #444;
  border-radius: 4px;
  background-color: #333;
  color: #ccc;
}

.pagination {
  margin-top: 20px;
}

.pagination button {
  padding: 10px;
  margin: 0 10px;
  background-color: #444;
  color: #ccc;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.pagination button:disabled {
  background-color: #555;
  cursor: not-allowed;
}

.pagination .active {
  background-color: #666;
}

.pagination span {
  color: #ccc;
  font-size: 1.2rem;
}

.album-card {
  display: inline-block;
  width: 220px;
  margin: 15px;
  text-align: center;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  overflow: hidden;
  background-color: #2c2c2c; /* Slightly lighter dark background for cards */
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.album-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.3);
}

.album-link {
  text-decoration: none;
  color: inherit;
}

.album-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
  border-bottom: 2px solid #444; /* Subtle border color */
}

.album-info {
  padding: 10px;
}

.album-info h2 {
  font-size: 1.25rem;
  color: #fff;
  margin: 10px 0;
  white-space: nowrap; /* Prevent the text from wrapping */
  overflow: hidden; /* Hide overflow */
  text-overflow: ellipsis; /* Add ellipsis for overflowing text */
}

.album-info p {
  font-size: 12px;
  color: #bbb; /* Lighter gray for artist name */
}

p {
  font-size: 1.2rem;
  color: #ccc;
}

.headerLeft {
  display: flex;
}
</style>
