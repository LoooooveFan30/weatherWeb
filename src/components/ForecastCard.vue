<script setup>
const props = defineProps({
  forecast: { type: Array, default: () => [] },
  unit: { type: String, default: 'C' }
})

const toFahrenheit = (c) => Math.round(c * 9 / 5 + 32)
const formatTemp = (c) => props.unit === 'F' ? toFahrenheit(c) : Math.round(c)

const weatherIcons = {
  sunny: '☀️', 'partly cloudy': '⛅', cloudy: '☁️',
  rainy: '🌧️', snowy: '❄️', stormy: '⛈️', foggy: '🌫️'
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
</script>

<template>
  <section class="forecast-card glass" v-if="forecast.length">
    <div class="forecast-title">未来预报</div>
    <div class="forecast-grid">
      <div class="forecast-item" v-for="day in forecast" :key="day.date">
        <div class="forecast-day">{{ getWeekday(day.date) }}</div>
        <div class="forecast-icon">{{ weatherIcons[day.condition] || '☀️' }}</div>
        <div class="forecast-temps">
          <span class="temp-high">{{ formatTemp(day.maxTemp) }}°</span>
          <span class="temp-divider">/</span>
          <span class="temp-low">{{ formatTemp(day.minTemp) }}°</span>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
.forecast-card {
  padding: 24px 28px;
  color: #fff;
  border-radius: 22px;
}

.forecast-title {
  font-size: 0.75rem;
  font-weight: 500;
  margin-bottom: 16px;
  color: rgba(255, 255, 255, 0.3);
  text-transform: uppercase;
  letter-spacing: 1.5px;
}

.forecast-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 6px;
}

.forecast-item {
  text-align: center;
  padding: 16px 6px 14px;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.03);
  transition: all 0.25s ease;
}

.forecast-item:hover {
  background: rgba(255, 255, 255, 0.07);
  transform: translateY(-2px);
}

.forecast-day {
  font-size: 0.72rem;
  color: rgba(255, 255, 255, 0.35);
  margin-bottom: 12px;
  font-weight: 500;
  letter-spacing: 0.3px;
}

.forecast-icon {
  font-size: 1.5rem;
  margin-bottom: 12px;
  filter: drop-shadow(0 2px 6px rgba(0, 0, 0, 0.1));
}

.forecast-temps {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 3px;
  font-size: 0.8rem;
}

.temp-high { font-weight: 700; }
.temp-divider { color: rgba(255, 255, 255, 0.12); font-weight: 300; }
.temp-low { font-weight: 400; color: rgba(255, 255, 255, 0.3); }

@media (max-width: 700px) {
  .forecast-grid { grid-template-columns: repeat(4, 1fr); }
}
@media (max-width: 400px) {
  .forecast-grid { grid-template-columns: repeat(3, 1fr); }
}
</style>
