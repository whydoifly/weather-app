<template>
  <div id="app" class="container">
    <h1>Прогноз погоды</h1>
    <div class="search-box">
      <input type="text" v-model="city" placeholder="Введите название города" @keyup.enter="getWeather" />
      <button @click="clearSearch" class="clear-btn">Очистить</button>
      <button @click="getWeather">Поиск</button>
    </div>
    <div v-if="error" class="error">{{ error }}</div>
    <div v-if="loading" class="loading">
      <i class="fas fa-spinner fa-spin"></i>
    </div>
    <div v-if="!loading && weather" class="weather-info">
      <h2>{{ weather.name }}, {{ weather.sys.country }}</h2>
      <i :class="getWeatherIcon(weather.weather[0].main)"></i>
      <p>
        Температура воздуха: <b>{{ convertTemp(weather.main.temp) }}</b> |
        <i class="fas fa-sync-alt toggle-icon" @click="toggleUnit"></i>
      </p>
      <p>Скорость ветра: <b>{{ weather.wind.speed }} м/с</b></p>
    </div>
    <div class="recent-searches">
      <h3>Недавний поиск:</h3>
      <ul>
        <li v-for="search in recentSearches" :key="search" @click="city = search; getWeather()">
          {{ search }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

const API_KEY = '9ec1236670f2e67edd63d254da39a060';
const BASE_URL = 'https://api.openweathermap.org/data/2.5/weather';

export default {
  data() {
    return {
      city: '',
      weather: null,
      error: null,
      unit: 'C',
      loading: false,
      recentSearches: [],
    };
  },
  methods: {
    async getWeather() {
      if (!this.city) return;

      // Убрать пробелы при наличии
      this.city = this.city.trim();

      // Состояние загрузки
      this.loading = true;

      try {
        const response = await axios.get(BASE_URL, {
          params: {
            q: this.city,
            appid: API_KEY,
            units: 'metric',
          },
        });

        this.weather = response.data;
        this.error = null;
        this.updateRecentSearches(this.city);
      } catch (error) {
        this.weather = null;
        // Проверка наличия спецсимволов
        if (/[.#$@!%^&*()]/g.test(this.city)) {
          this.error = 'Пожалуйста введите правильное название города без посторонних символов.';
        } else {
          this.error = 'Город не найден. Пожалуйста, попробуйте ещё раз.';
        }
      } finally {
        this.loading = false;
      }
    },
    getWeatherIcon(condition) {
      const iconMap = {
        'Clear': 'fas fa-sun',
        'Clouds': 'fas fa-cloud',
        'Rain': 'fas fa-cloud-rain',
        'Drizzle': 'fas fa-cloud-rain',
        'Thunderstorm': 'fas fa-bolt',
        'Snow': 'far fa-snowflake',
        'Mist': 'fas fa-smog',
      };
      
      return iconMap[condition] || 'fas fa-question';
    },
    convertTemp(temp) {
      if (this.unit === 'C') {
        return `${temp.toFixed(1)}°C`;
      } else {
        return `${((temp * 9) / 5 + 32).toFixed(1)}°F`;
      }
    },
    toggleUnit() {
      this.unit = this.unit === 'C' ? 'F' : 'C';
    },
    clearSearch() {
      this.city = '';
      this.weather = null;
      this.error = null;
    },
    updateRecentSearches(city) {
      if (!this.recentSearches.includes(city)) {
        this.recentSearches.unshift(city);
        if (this.recentSearches.length > 5) {
          this.recentSearches.pop();
        }
      }
    },
  },
};
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: "Raleway", sans-serif;
  font-optical-sizing: auto;
  font-weight: 400;
  font-style: normal;
}

.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

.loading i {
  font-size: 24px;
}

.search-box {
  display: flex;
  margin-bottom: 20px;
  border: 1px solid #D1E9F6;
  padding: 0 0 0 10px;
  border-radius: 10px;
}

.search-box input {
  flex: 1;
  font-size: 16px;
  outline: none;
}

.search-box button {
  padding: 10px 20px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  border-radius: 10px;
  transition: all 0.3s ease-in-out;
}

.search-box .clear-btn {
  background-color: transparent;
}

.search-box .clear-btn:hover {
  color: #EECAD5;
  background-color: transparent;
}

.search-box button:hover {
  background-color: #D1E9F6;
}

.search-box input,
.search-box button {
  border: none;
}

.weather-info {
  margin-bottom: 20px;
}

.weather-info i {
  font-size: 48px;
  margin-bottom: 10px;
}

.weather-info .toggle-icon {
  font-size: 16px;
  margin-left: 5px;
  cursor: pointer;
}

.weather-info p {
  margin: 5px 0;
  font-size: 20px;
}

.weather-info span {
  cursor: pointer;
}

.recent-searches ul {
  list-style-type: none;
  padding: 0;
}

.recent-searches li {
  margin-bottom: 5px;
  cursor: pointer;
  background-color: #D1E9F6;
  border-radius: 10px;
  padding: 10px;
  transition: all 0.3s ease-in-out;
}

.recent-searches li:hover {
  background-color: #51aee0;
}

.error {
  color: red;
  margin-bottom: 10px;
}

@media (max-width: 768px) {
  h1 {
    font-size: 26px;
  }

  .search-box {
    flex-direction: column;
    padding: 10px 0 0 0;
  }

  .search-box input {
    padding: 0 10px 10px;
    text-align: center;
  }

  .weather-info p {
    font-size: 16px;
  }

  .search-box button {
    border-radius: 0;
  }

  .search-box .clear-btn {
    order: 2;
  }
}
</style>