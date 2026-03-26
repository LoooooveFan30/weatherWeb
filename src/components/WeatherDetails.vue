<script setup>
import { computed } from 'vue'

const props = defineProps({
  weatherData: {
    type: Object,
    default: null
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

const tempUnit = computed(() => props.unit === 'F' ? '°F' : '°C')

const getUvLevel = (uv) => {
  if (uv <= 2) return { text: '低', color: '#4ade80' }
  if (uv <= 5) return { text: '中', color: '#facc15' }
  if (uv <= 7) return { text: '高', color: '#f97316' }
  if (uv <= 10) return { text: '极高', color: '#ef4444' }
  return { text: '危险', color: '#a855f7' }
}

const getWindDirection = (deg) => {
  if (deg === undefined || deg === null) return ''
  const dirs = ['北', '东北', '东', '东南', '南', '西南', '西', '西北']
  return dirs[Math.round(deg / 45) % 8]
}
</script>

<template>
  <section class="detail-grid" v-if="weatherData">
    <div class="detail-card glass">
      <div class="detail-icon">💧</div>
      <div class="detail-label">湿度</div>
      <div class="detail-value">{{ weatherData.humidity }}<small>%</small></div>
      <div class="detail-bar">
        <div class="detail-bar-fill" :style="{ width: weatherData.humidity + '%' }"></div>
      </div>
    </div>
    <div class="detail-card glass">
      <div class="detail-icon">💨</div>
      <div class="detail-label">风速</div>
      <div class="detail-value">{{ weatherData.windSpeed }}<small>km/h</small></div>
      <div class="detail-sub" v-if="weatherData.windDirection !== undefined">
        {{ getWindDirection(weatherData.windDirection) }}风
      </div>
    </div>
    <div class="detail-card glass">
      <div class="detail-icon">🌡️</div>
      <div class="detail-label">气压</div>
      <div class="detail-value">{{ weatherData.pressure }}<small>hPa</small></div>
    </div>
    <div class="detail-card glass">
      <div class="detail-icon">☀️</div>
      <div class="detail-label">紫外线</div>
      <div class="detail-value" :style="{ color: getUvLevel(weatherData.uvIndex || 0).color }">
        {{ Math.round(weatherData.uvIndex || 0) }}
      </div>
      <div class="detail-sub" :style="{ color: getUvLevel(weatherData.uvIndex || 0).color }">
        {{ getUvLevel(weatherData.uvIndex || 0).text }}
      </div>
    </div>
    <div class="detail-card glass">
      <div class="detail-icon">☁️</div>
      <div class="detail-label">云量</div>
      <div class="detail-value">{{ weatherData.cloudCover }}<small>%</small></div>
      <div class="detail-bar">
        <div class="detail-bar-fill" :style="{ width: weatherData.cloudCover + '%' }"></div>
      </div>
    </div>
  </section>
</template>

<style scoped>
.detail-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 12px;
}

.detail-card {
  padding: 18px 14px;
  color: #fff;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.detail-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.2);
}

.detail-icon {
  font-size: 1.5rem;
}

.detail-label {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.65);
  text-transform: uppercase;
  letter-spacing: 1px;
}

.detail-value {
  font-size: 1.5rem;
  font-weight: 700;
}

.detail-value small {
  font-size: 0.65rem;
  font-weight: 400;
  margin-left: 2px;
  color: rgba(255, 255, 255, 0.7);
}

.detail-sub {
  font-size: 0.7rem;
  color: rgba(255, 255, 255, 0.5);
}

.detail-bar {
  width: 100%;
  height: 4px;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 2px;
  overflow: hidden;
  margin-top: 4px;
}

.detail-bar-fill {
  height: 100%;
  background: linear-gradient(90deg, rgba(255, 255, 255, 0.4), rgba(255, 255, 255, 0.9));
  border-radius: 2px;
  transition: width 1s ease;
}

@media (max-width: 700px) {
  .detail-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
