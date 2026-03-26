<script setup>
import { computed } from 'vue'

const props = defineProps({
  forecast: {
    type: Array,
    default: () => []
  },
  unit: {
    type: String,
    default: 'C'
  }
})

const toFahrenheit = (c) => Math.round(c * 9 / 5 + 32)

const formatTemp = (celsius) => {
  if (props.unit === 'F') return toFahrenheit(celsius)
  return Math.round(celsius)
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

const weatherTexts = {
  sunny: '晴',
  'partly cloudy': '多云',
  cloudy: '阴',
  rainy: '雨',
  snowy: '雪',
  stormy: '雷',
  foggy: '雾'
}

const weekdays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六']

const getWeekday = (dateStr) => {
  const d = new Date(dateStr)
  const today = new Date()
  if (d.toDateString() === today.toDateString()) return '今天'
  const tomorrow = new Date(today)
  tomorrow.setDate(today.getDate() + 1)
  if (d.toDateString() === tomorrow.toDateString()) return '明天'
  return weekdays[d.getDay()]
}

const formatDate = (dateStr) => {
  const d = new Date(dateStr)
  return `${d.getMonth() + 1}/${d.getDate()}`
}

const getUvLevel = (uv) => {
  if (uv <= 2) return { text: '低', color: '#4ade80' }
  if (uv <= 5) return { text: '中', color: '#facc15' }
  if (uv <= 7) return { text: '高', color: '#f97316' }
  if (uv <= 10) return { text: '极高', color: '#ef4444' }
  return { text: '危险', color: '#a855f7' }
}
</script>

<template>
  <section class="forecast-card glass" v-if="forecast.length">
    <div class="forecast-title">未来天气预报</div>
    <div class="forecast-grid">
      <div
        class="forecast-item"
        v-for="day in forecast"
        :key="day.date"
      >
        <div class="forecast-day">{{ getWeekday(day.date) }}</div>
        <div class="forecast-date">{{ formatDate(day.date) }}</div>
        <div class="forecast-icon">{{ weatherIcons[day.condition] || '☀️' }}</div>
        <div class="forecast-condition">{{ weatherTexts[day.condition] || '晴' }}</div>
        <div class="forecast-temps">
          <span class="temp-high">{{ formatTemp(day.maxTemp) }}°</span>
          <span class="temp-divider">/</span>
          <span class="temp-low">{{ formatTemp(day.minTemp) }}°</span>
        </div>
        <div class="forecast-details" v-if="day.uvIndex !== undefined || day.precipitation !== undefined">
          <div class="forecast-detail" v-if="day.uvIndex !== undefined" title="紫外线指数">
            <span class="detail-icon">☀️</span>
            <span class="detail-value" :style="{ color: getUvLevel(day.uvIndex).color }">{{ day.uvIndex }}</span>
          </div>
          <div class="forecast-detail" v-if="day.precipitation > 0" title="降水量">
            <span class="detail-icon">💧</span>
            <span class="detail-value">{{ day.precipitation.toFixed(1) }}mm</span>
          </div>
          <div class="forecast-detail" v-if="day.windSpeed > 0" title="最大风速">
            <span class="detail-icon">💨</span>
            <span class="detail-value">{{ day.windSpeed }}km/h</span>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
.forecast-card {
  padding: 24px 28px;
  color: #fff;
}

.forecast-title {
  font-size: 1rem;
  font-weight: 600;
  margin-bottom: 16px;
  color: rgba(255, 255, 255, 0.85);
}

.forecast-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 8px;
}

.forecast-item {
  text-align: center;
  padding: 14px 8px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.06);
  transition: background 0.2s, transform 0.2s;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.forecast-item:hover {
  background: rgba(255, 255, 255, 0.12);
  transform: translateY(-2px);
}

.forecast-day {
  font-size: 0.85rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.9);
}

.forecast-date {
  font-size: 0.7rem;
  color: rgba(255, 255, 255, 0.4);
}

.forecast-icon {
  font-size: 2rem;
  margin: 4px 0;
}

.forecast-condition {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.6);
}

.forecast-temps {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  margin-top: 4px;
}

.temp-high {
  font-weight: 700;
  font-size: 1rem;
}

.temp-divider {
  color: rgba(255, 255, 255, 0.3);
  font-size: 0.8rem;
}

.temp-low {
  font-weight: 400;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.5);
}

.forecast-details {
  display: flex;
  flex-direction: column;
  gap: 2px;
  margin-top: 6px;
  padding-top: 6px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.forecast-detail {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.65rem;
}

.detail-icon {
  font-size: 0.7rem;
}

.detail-value {
  color: rgba(255, 255, 255, 0.7);
}

@media (max-width: 900px) {
  .forecast-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}

@media (max-width: 700px) {
  .forecast-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 400px) {
  .forecast-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
