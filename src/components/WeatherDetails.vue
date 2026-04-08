<script setup>
import { computed } from 'vue'

const props = defineProps({
  weatherData: { type: Object, default: null },
  unit: { type: String, default: 'C' }
})

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
  padding: 20px 14px;
  color: #fff;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  transition: all 0.3s ease;
  border-radius: 20px;
}

.detail-card:hover {
  transform: translateY(-3px);
  border-color: rgba(255, 255, 255, 0.15);
}

.detail-icon {
  font-size: 1.3rem;
  opacity: 0.8;
}

.detail-label {
  font-size: 0.68rem;
  color: rgba(255, 255, 255, 0.3);
  text-transform: uppercase;
  letter-spacing: 1.2px;
  font-weight: 500;
}

.detail-value {
  font-size: 1.4rem;
  font-weight: 700;
  letter-spacing: -0.5px;
}

.detail-value small {
  font-size: 0.6rem;
  font-weight: 400;
  margin-left: 2px;
  color: rgba(255, 255, 255, 0.35);
}

.detail-sub {
  font-size: 0.68rem;
  color: rgba(255, 255, 255, 0.4);
  font-weight: 400;
}

.detail-bar {
  width: 100%;
  height: 3px;
  background: rgba(255, 255, 255, 0.06);
  border-radius: 2px;
  overflow: hidden;
  margin-top: 4px;
}

.detail-bar-fill {
  height: 100%;
  background: linear-gradient(90deg, rgba(255, 255, 255, 0.12), rgba(255, 255, 255, 0.4));
  border-radius: 2px;
  transition: width 1s ease;
}

@media (max-width: 700px) {
  .detail-grid { grid-template-columns: repeat(2, 1fr); gap: 10px; }
}
</style>
