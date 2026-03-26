<script setup>
import { ref, onMounted, computed } from 'vue'
import WeatherAnimation from './components/WeatherAnimation.vue'
import ClockDisplay from './components/ClockDisplay.vue'
import SearchBar from './components/SearchBar.vue'
import WeatherDetails from './components/WeatherDetails.vue'
import NetworkInfo from './components/NetworkInfo.vue'
import ForecastCard from './components/ForecastCard.vue'
import { OPEN_METEO_BASE, GEOCODING_BASE } from './config.js'

// ===== 响应式状态 =====
const locationData = ref(null)
const weatherData = ref(null)
const forecastData = ref([])
const hourlyData = ref([])
const loading = ref(true)
const error = ref(null)
const tempUnit = ref('C')
const CACHE_KEY = 'weather-app-cache'

// ===== 搜索弹窗状态 =====
const showModal = ref(false)
const searchResults = ref([])
const searching = ref(false)
const visitCount = ref(0)

// ===== WMO 天气代码映射 =====
const wmoCodeToCondition = (code) => {
  if (code === 0) return 'sunny'
  if (code === 1 || code === 2) return 'partly cloudy'
  if (code === 3) return 'cloudy'
  if (code >= 45 && code <= 48) return 'foggy'
  if (code >= 51 && code <= 55) return 'rainy'
  if (code >= 56 && code <= 57) return 'rainy'
  if (code >= 61 && code <= 65) return 'rainy'
  if (code >= 66 && code <= 67) return 'rainy'
  if (code >= 71 && code <= 77) return 'snowy'
  if (code >= 80 && code <= 82) return 'rainy'
  if (code >= 85 && code <= 86) return 'snowy'
  if (code >= 95 && code <= 99) return 'stormy'
  return 'sunny'
}

const wmoCodeToText = (code) => {
  const texts = {
    0: '晴天',
    1: '大部晴朗',
    2: '局部多云',
    3: '阴天',
    45: '雾',
    48: '雾凇',
    51: '小毛毛雨',
    53: '中毛毛雨',
    55: '大毛毛雨',
    56: '冻毛毛雨',
    57: '强冻毛毛雨',
    61: '小雨',
    63: '中雨',
    65: '大雨',
    66: '冻雨',
    67: '强冻雨',
    71: '小雪',
    73: '中雪',
    75: '大雪',
    77: '米雪',
    80: '小阵雨',
    81: '中阵雨',
    82: '大阵雨',
    85: '小阵雪',
    86: '大阵雪',
    95: '雷暴',
    96: '雷暴伴小冰雹',
    99: '雷暴伴大冰雹'
  }
  return texts[code] || '未知'
}

const weatherIcons = {
  sunny: '☀️',
  'partly cloudy': '⛅',
  cloudy: '☁️',
  rainy: '🌧️',
  snowy: '❄️',
  stormy: '⛈️',
  foggy: '🌫️'
}

// ===== Computed =====

const bgWeatherType = computed(() => {
  if (!weatherData.value) return 'clear'
  const map = {
    sunny: 'clear',
    'partly cloudy': 'cloudy',
    cloudy: 'overcast',
    rainy: 'rain',
    snowy: 'snow',
    stormy: 'thunder',
    foggy: 'fog'
  }
  return map[weatherData.value.condition] || 'clear'
})

const isNight = computed(() => {
  const hour = new Date().getHours()
  return hour >= 18 || hour < 6
})

const greeting = computed(() => {
  const hour = new Date().getHours()
  if (hour < 6) return '夜深了'
  if (hour < 12) return '早上好'
  if (hour < 18) return '下午好'
  return '晚上好'
})

const formattedDate = computed(() => {
  return new Date().toLocaleDateString('zh-CN', {
    year: 'numeric', month: 'long', day: 'numeric', weekday: 'long'
  })
})

const tempUnitSymbol = computed(() => tempUnit.value === 'F' ? '°F' : '°C')

const displayTemp = computed(() => {
  if (!weatherData.value) return 0
  return tempUnit.value === 'F'
    ? Math.round(weatherData.value.temp * 9 / 5 + 32)
    : weatherData.value.temp
})

const displayFeelsLike = computed(() => {
  if (!weatherData.value) return 0
  return tempUnit.value === 'F'
    ? Math.round(weatherData.value.feelsLike * 9 / 5 + 32)
    : weatherData.value.feelsLike
})

const toggleUnit = () => {
  tempUnit.value = tempUnit.value === 'C' ? 'F' : 'C'
}

// ===== 缓存 =====

const saveCache = (key, data) => {
  try {
    localStorage.setItem(`${CACHE_KEY}-${key}`, JSON.stringify({ timestamp: Date.now(), data }))
  } catch { /* 忽略 */ }
}

const loadCache = (key, maxAge = 10 * 60 * 1000) => {
  try {
    const raw = localStorage.getItem(`${CACHE_KEY}-${key}`)
    if (!raw) return null
    const cache = JSON.parse(raw)
    if (Date.now() - cache.timestamp > maxAge) return null
    return cache.data
  } catch { return null }
}

// ===== 带超时的 fetch =====

const fetchJSON = async (url, timeout = 10000) => {
  const controller = new AbortController()
  const timer = setTimeout(() => controller.abort(), timeout)
  try {
    const res = await fetch(url, { signal: controller.signal })
    clearTimeout(timer)
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    const text = await res.text()
    if (!text || text.trim() === '') throw new Error('Empty response')
    return JSON.parse(text)
  } catch (e) {
    clearTimeout(timer)
    throw e
  }
}

// ===== IP 定位 =====

const fetchLocation = async () => {
  try {
    // 检查缓存
    const cached = loadCache('location', 30 * 60 * 1000)
    if (cached) {
      locationData.value = cached
      await fetchWeatherByCoords(cached.lat, cached.lon, cached.city, cached.region, cached.country)
      return
    }

    let loc = null

    // 方案1：ip-api.com
    try {
      const d = await fetchJSON('http://ip-api.com/json/?lang=zh-CN', 8000)
      if (d && d.status === 'success') {
        loc = {
          city: d.city || '未知',
          region: d.regionName || '未知',
          country: d.country || '未知',
          ip: d.query || '未知',
          isp: d.isp || '未知',
          timezone: d.timezone || '未知',
          lat: d.lat,
          lon: d.lon
        }
      }
    } catch (e) { console.warn('[IP定位] ip-api 失败:', e.message) }

    // 方案2：ipinfo.io
    if (!loc) {
      try {
        const d = await fetchJSON('https://ipinfo.io/json', 8000)
        if (d && d.ip) {
          const [lat, lon] = (d.loc || '39.9,116.4').split(',').map(Number)
          loc = {
            city: d.city || '未知',
            region: d.region || '未知',
            country: d.country || '未知',
            ip: d.ip || '未知',
            isp: d.org || '未知',
            timezone: d.timezone || '未知',
            lat,
            lon
          }
        }
      } catch (e) { console.warn('[IP定位] ipinfo 失败:', e.message) }
    }

    if (!loc) {
      error.value = '获取位置信息失败，请搜索城市查看天气'
      loading.value = false
      return
    }

    locationData.value = loc
    saveCache('location', loc)
    await fetchWeatherByCoords(loc.lat, loc.lon, loc.city, loc.region, loc.country)
  } catch (e) {
    console.error('[fetchLocation] 异常:', e)
    error.value = '获取位置信息失败，请搜索城市查看天气'
    loading.value = false
  }
}

// ===== Open-Meteo：通过经纬度获取天气 =====

const fetchWeatherByCoords = async (lat, lon, cityName, regionName, countryName) => {
  try {
    // 检查缓存
    const cacheKey = `weather-${lat.toFixed(2)}-${lon.toFixed(2)}`
    const cached = loadCache(cacheKey)
    if (cached) {
      weatherData.value = cached.current
      forecastData.value = cached.forecast
      hourlyData.value = cached.hourly || []
      loading.value = false
      return
    }

    // Open-Meteo API 请求
    const params = new URLSearchParams({
      latitude: lat,
      longitude: lon,
      current_weather: true,
      hourly: 'temperature_2m,relativehumidity_2m,apparent_temperature,weathercode,cloudcover,pressure_msl,windspeed_10m,uv_index',
      daily: 'weathercode,temperature_2m_max,temperature_2m_min,uv_index_max,precipitation_sum,windspeed_10m_max',
      timezone: 'auto',
      forecast_days: 7
    })

    const url = `${OPEN_METEO_BASE}/forecast?${params}`
    const data = await fetchJSON(url)

    // 解析当前天气
    const current = data?.current_weather
    if (!current) throw new Error('天气数据为空')

    const condition = wmoCodeToCondition(current.weathercode)
    weatherData.value = {
      temp: Math.round(current.temperature),
      feelsLike: Math.round(getFeelsLike(current.temperature, data.hourly?.relativehumidity_2m?.[0] || 50, current.windspeed)),
      humidity: data.hourly?.relativehumidity_2m?.[0] || 0,
      windSpeed: Math.round(current.windspeed),
      windDirection: current.winddirection,
      pressure: Math.round(data.hourly?.pressure_msl?.[0] || 1013),
      cloudCover: data.hourly?.cloudcover?.[0] || 0,
      uvIndex: data.hourly?.uv_index?.[0] || 0,
      condition,
      icon: weatherIcons[condition] || '☀️',
      text: wmoCodeToText(current.weathercode),
      isDay: current.is_day === 1
    }

    // 解析多日预报
    const daily = data?.daily
    if (daily && daily.time) {
      forecastData.value = daily.time.map((date, i) => ({
        date,
        condition: wmoCodeToCondition(daily.weathercode[i]),
        maxTemp: Math.round(daily.temperature_2m_max[i]),
        minTemp: Math.round(daily.temperature_2m_min[i]),
        uvIndex: Math.round(daily.uv_index_max[i] || 0),
        precipitation: daily.precipitation_sum?.[i] || 0,
        windSpeed: Math.round(daily.windspeed_10m_max?.[i] || 0)
      }))
    }

    // 解析小时预报
    const hourly = data?.hourly
    if (hourly && hourly.time) {
      const now = new Date()
      hourlyData.value = hourly.time
        .map((time, i) => ({
          time,
          temp: Math.round(hourly.temperature_2m[i]),
          condition: wmoCodeToCondition(hourly.weathercode[i]),
          humidity: hourly.relativehumidity_2m[i],
          windSpeed: Math.round(hourly.windspeed_10m[i]),
          uvIndex: hourly.uv_index?.[i] || 0
        }))
        .filter(h => new Date(h.time) >= now)
        .slice(0, 24)
    }

    // 更新位置信息（补充经纬度）
    if (locationData.value) {
      locationData.value.lat = lat
      locationData.value.lon = lon
    } else {
      locationData.value = {
        city: cityName || '未知',
        region: regionName || '未知',
        country: countryName || '未知',
        ip: '未知',
        isp: '未知',
        timezone: data.timezone || '未知',
        lat,
        lon
      }
    }

    // 保存缓存
    saveCache(cacheKey, { current: weatherData.value, forecast: forecastData.value, hourly: hourlyData.value })
  } catch (e) {
    console.error('[fetchWeatherByCoords] 异常:', e)
    error.value = '获取天气信息失败，请稍后重试'
  }
  loading.value = false
}

// ===== 体感温度计算 =====
const getFeelsLike = (temp, humidity, windSpeed) => {
  // 风寒指数（低温时）
  if (temp <= 10 && windSpeed > 4.8) {
    return 13.12 + 0.6215 * temp - 11.37 * Math.pow(windSpeed, 0.16) + 0.3965 * temp * Math.pow(windSpeed, 0.16)
  }
  // 热指数（高温时）
  if (temp >= 27) {
    const c1 = -8.78469475556
    const c2 = 1.61139411
    const c3 = 2.33854883889
    const c4 = -0.14611605
    const c5 = -0.012308094
    const c6 = -0.0164248277778
    const c7 = 0.002211732
    const c8 = 0.00072546
    const c9 = -0.000003582
    return c1 + c2 * temp + c3 * humidity + c4 * temp * humidity + c5 * temp * temp + c6 * humidity * humidity + c7 * temp * temp * humidity + c8 * temp * humidity * humidity + c9 * temp * temp * humidity * humidity
  }
  return temp
}

// ===== 城市搜索（使用 Open-Meteo Geocoding API） =====

const handleSearch = async (query) => {
  searching.value = true
  error.value = null

  try {
    // 使用 Open-Meteo 地理编码 API 搜索城市
    const url = `${GEOCODING_BASE}/search?name=${encodeURIComponent(query)}&count=10&language=zh&format=json`
    const data = await fetchJSON(url)

    if (!data.results || data.results.length === 0) {
      error.value = '未找到该城市，请尝试其他名称'
      searching.value = false
      return
    }

    if (data.results.length === 1) {
      // 只有一个结果，直接选择
      await selectCity(data.results[0])
    } else {
      // 多个结果，显示选择弹窗
      searchResults.value = data.results.map(r => ({
        id: r.id,
        name: r.name,
        lat: r.latitude,
        lon: r.longitude,
        country: r.country || '',
        country_code: r.country_code || '',
        admin1: r.admin1 || '',
        admin2: r.admin2 || '',
        timezone: r.timezone || ''
      }))
      showModal.value = true
    }
  } catch (e) {
    console.error('[handleSearch] 异常:', e)
    error.value = '搜索失败，请检查网络连接'
    loading.value = false
  }

  searching.value = false
}

// ===== 用户选择城市 =====

const selectCity = async (city) => {
  showModal.value = false
  loading.value = true
  error.value = null

  try {
    locationData.value = {
      city: city.name,
      region: city.admin1 || '未知',
      country: city.country || '未知',
      ip: locationData.value?.ip || '未知',
      isp: locationData.value?.isp || '未知',
      timezone: city.timezone || '未知',
      lat: city.lat,
      lon: city.lon
    }

    // 清除缓存，强制获取最新天气
    const cacheKey = `weather-${city.lat.toFixed(2)}-${city.lon.toFixed(2)}`
    localStorage.removeItem(`${CACHE_KEY}-${cacheKey}`)

    await fetchWeatherByCoords(city.lat, city.lon, city.name, city.admin1, city.country)
  } catch (e) {
    console.error('[selectCity] 异常:', e)
    error.value = '获取天气信息失败，请重试'
    loading.value = false
  }
}

// ===== 关闭弹窗 =====

const closeModal = () => {
  showModal.value = false
  if (!weatherData.value) {
    loading.value = false
  }
}

// ===== 手动刷新 =====

const refreshWeather = async () => {
  loading.value = true
  error.value = null

  try {
    // 清除所有缓存
    localStorage.removeItem(`${CACHE_KEY}-location`)
    Object.keys(localStorage).forEach(key => {
      if (key.startsWith(`${CACHE_KEY}-weather-`)) {
        localStorage.removeItem(key)
      }
    })
    await fetchLocation()
  } catch (e) {
    error.value = '刷新失败，请重试'
    loading.value = false
  }
}

// ===== 生命周期 =====

onMounted(() => {
  // 统计访问量
  const count = parseInt(localStorage.getItem('weather-visit-count') || '0') + 1
  localStorage.setItem('weather-visit-count', count.toString())
  visitCount.value = count

  fetchLocation()
})
</script>

<template>
  <div class="app-root">
    <WeatherAnimation :weather="bgWeatherType" :isNight="isNight" />

    <!-- 加载状态 -->
    <div class="loading-screen" v-if="loading">
      <div class="loader-ring">
        <div class="loader-arc"></div>
      </div>
      <p class="loading-text">正在获取天气数据...</p>
    </div>

    <!-- 错误状态 -->
    <div class="error-screen" v-else-if="error">
      <div class="error-card glass">
        <span class="error-icon">⚠️</span>
        <p>{{ error }}</p>
        <div class="error-actions">
          <button class="retry-btn" @click="refreshWeather">重试</button>
        </div>
      </div>
    </div>

    <!-- 主内容 -->
    <div class="main-content" v-else>
      <SearchBar @search="handleSearch" />

      <!-- 搜索中提示 -->
      <div class="searching-indicator" v-if="searching">
        <div class="searching-dot"></div>
        <span>正在搜索城市...</span>
      </div>

      <!-- 顶部栏 -->
      <header class="top-bar glass">
        <div class="brand">
          <span class="brand-icon">🌤️</span>
          <span class="brand-name">Weather Now</span>
        </div>
        <div class="top-bar-right">
          <button class="unit-btn" @click="toggleUnit" :title="tempUnit === 'C' ? '切换到华氏度' : '切换到摄氏度'">
            °{{ tempUnit === 'C' ? 'C' : 'F' }}
          </button>
          <button class="refresh-btn" @click="refreshWeather" title="刷新天气">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M21 2v6h-6"/><path d="M3 12a9 9 0 0 1 15-6.7L21 8"/>
              <path d="M3 22v-6h6"/><path d="M21 12a9 9 0 0 1-15 6.7L3 16"/>
            </svg>
          </button>
          <ClockDisplay />
        </div>
      </header>

      <!-- 核心天气卡片 -->
      <section class="hero-card glass">
        <div class="hero-left">
          <div class="greeting">{{ greeting }}</div>
          <div class="location-row">
            <svg class="pin-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"/>
              <circle cx="12" cy="10" r="3"/>
            </svg>
            <span class="city">{{ locationData?.city }}</span>
            <span class="region">{{ locationData?.region }}, {{ locationData?.country }}</span>
          </div>
          <div class="date-info">{{ formattedDate }}</div>
        </div>
        <div class="hero-right">
          <div class="weather-icon-big">{{ weatherData?.icon }}</div>
          <div class="temp-main">{{ displayTemp }}<span class="degree">{{ tempUnitSymbol }}</span></div>
          <div class="condition-label">{{ weatherData?.text }}</div>
          <div class="feels-like">体感 {{ displayFeelsLike }}{{ tempUnitSymbol }}</div>
        </div>
      </section>

      <WeatherDetails :weatherData="weatherData" :unit="tempUnit" />
      <ForecastCard :forecast="forecastData" :unit="tempUnit" />
      <NetworkInfo :locationData="locationData" />

      <footer class="footer">
        <span>数据来源: Open-Meteo</span>
        <span class="visit-count">👁️ 访问量: {{ visitCount }}</span>
      </footer>
    </div>

    <!-- 城市选择弹窗 -->
    <Teleport to="body">
      <div class="modal-overlay" v-if="showModal" @click.self="closeModal">
        <div class="modal-box glass">
          <div class="modal-header">
            <span class="modal-title">选择城市</span>
            <button class="modal-close" @click="closeModal">✕</button>
          </div>
          <p class="modal-hint">找到 {{ searchResults.length }} 个匹配结果：</p>
          <div class="modal-list">
            <button
              class="modal-item"
              v-for="city in searchResults"
              :key="city.id"
              @click="selectCity(city)"
            >
              <span class="modal-item-name">{{ city.name }}</span>
              <span class="modal-item-detail">
                {{ city.admin1 ? city.admin1 + ', ' : '' }}{{ city.country }}
              </span>
            </button>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>

<style>
/* ===== 全局 ===== */
.app-root {
  position: relative;
  min-height: 100vh;
  overflow-x: hidden;
}

.glass {
  background: rgba(255, 255, 255, 0.12);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  border: 1px solid rgba(255, 255, 255, 0.25);
  border-radius: 20px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.12);
}

/* ===== 主内容布局 ===== */
.main-content {
  position: relative;
  z-index: 10;
  max-width: 900px;
  margin: 0 auto;
  padding: 24px 20px 40px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  min-height: 100vh;
}

/* ===== 搜索中指示器 ===== */
.searching-indicator {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.85rem;
}

.searching-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #fff;
  animation: pulse 1s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 0.3; transform: scale(0.8); }
  50% { opacity: 1; transform: scale(1.2); }
}

/* ===== 顶部栏 ===== */
.top-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 24px;
  color: #fff;
}

.brand {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 1.15rem;
  font-weight: 700;
}

.brand-icon { font-size: 1.5rem; }

.top-bar-right {
  display: flex;
  align-items: center;
  gap: 12px;
}

.unit-btn {
  padding: 4px 10px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  font-size: 0.85rem;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.2s;
}
.unit-btn:hover { background: rgba(255, 255, 255, 0.25); }

.refresh-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border: none;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  cursor: pointer;
  transition: background 0.2s, transform 0.3s;
}
.refresh-btn:hover {
  background: rgba(255, 255, 255, 0.25);
  transform: rotate(180deg);
}

/* ===== 天气卡片 ===== */
.hero-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 36px 40px;
  color: #fff;
  min-height: 220px;
}

.hero-left {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.greeting {
  font-size: 1.1rem;
  color: rgba(255, 255, 255, 0.75);
}

.location-row {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.pin-icon {
  width: 20px;
  height: 20px;
  color: #ff6b6b;
  flex-shrink: 0;
}

.city {
  font-size: 1.8rem;
  font-weight: 700;
}

.region {
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.7);
}

.date-info {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.6);
}

.hero-right {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}

.weather-icon-big {
  font-size: 4.5rem;
  line-height: 1;
  filter: drop-shadow(0 4px 12px rgba(0, 0, 0, 0.2));
  animation: iconFloat 3s ease-in-out infinite;
}

@media (prefers-reduced-motion: reduce) {
  .weather-icon-big, .searching-dot, .loader-arc { animation: none; }
}

@keyframes iconFloat {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}

.temp-main {
  font-size: 4rem;
  font-weight: 800;
  letter-spacing: -2px;
}

.degree {
  font-size: 1.8rem;
  font-weight: 400;
  vertical-align: super;
}

.condition-label {
  font-size: 1.1rem;
  color: rgba(255, 255, 255, 0.85);
}

.feels-like {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.6);
}

/* ===== 底部 ===== */
.footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  text-align: center;
  padding: 16px 0;
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.4);
}

.visit-count {
  font-size: 0.75rem;
}

/* ===== 加载页 ===== */
.loading-screen {
  position: fixed;
  inset: 0;
  z-index: 100;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 24px;
}

.loader-ring { width: 64px; height: 64px; }

.loader-arc {
  width: 100%;
  height: 100%;
  border: 4px solid rgba(255, 255, 255, 0.15);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.9s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }

.loading-text {
  color: rgba(255, 255, 255, 0.8);
  font-size: 1rem;
}

/* ===== 错误页 ===== */
.error-screen {
  position: fixed;
  inset: 0;
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
}

.error-card {
  padding: 40px 48px;
  text-align: center;
  color: #fff;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  max-width: 90vw;
}

.error-icon { font-size: 3rem; }

.error-card p {
  max-width: 320px;
  line-height: 1.6;
}

.error-actions {
  display: flex;
  gap: 12px;
  margin-top: 8px;
}

.retry-btn {
  padding: 10px 32px;
  border: none;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.2);
  color: #fff;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}
.retry-btn:hover { background: rgba(255, 255, 255, 0.35); }

/* ===== 城市选择弹窗 ===== */
.modal-overlay {
  position: fixed;
  inset: 0;
  z-index: 200;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.modal-box {
  width: 100%;
  max-width: 420px;
  max-height: 70vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 24px 12px;
  color: #fff;
}

.modal-title {
  font-size: 1.1rem;
  font-weight: 700;
}

.modal-close {
  width: 28px;
  height: 28px;
  border: none;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.12);
  color: #fff;
  font-size: 0.9rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
}
.modal-close:hover { background: rgba(255, 255, 255, 0.25); }

.modal-hint {
  padding: 0 24px 12px;
  color: rgba(255, 255, 255, 0.55);
  font-size: 0.85rem;
}

.modal-list {
  overflow-y: auto;
  padding: 0 12px 12px;
}

.modal-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 14px 16px;
  border: none;
  border-radius: 12px;
  background: transparent;
  color: #fff;
  font-size: 0.95rem;
  cursor: pointer;
  text-align: left;
  transition: background 0.15s;
}
.modal-item:hover { background: rgba(255, 255, 255, 0.1); }

.modal-item-name { font-weight: 600; }

.modal-item-detail {
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.5);
}

/* ===== 响应式 ===== */
@media (max-width: 700px) {
  .main-content { padding: 16px 12px 32px; }

  .hero-card {
    flex-direction: column;
    text-align: center;
    padding: 28px 20px;
    gap: 20px;
  }

  .hero-left { align-items: center; }
  .location-row { justify-content: center; }
  .temp-main { font-size: 3.2rem; }
  .weather-icon-big { font-size: 3.5rem; }
}
</style>
