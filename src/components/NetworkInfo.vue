<script setup>
defineProps({
  locationData: { type: Object, default: null }
})
</script>

<template>
  <section class="ip-card glass" v-if="locationData">
    <div class="ip-header">
      <svg class="ip-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
        <rect x="2" y="3" width="20" height="14" rx="2"/>
        <path d="M8 21h8M12 17v4"/>
      </svg>
      <span>网络信息</span>
    </div>
    <div class="ip-grid">
      <div class="ip-item">
        <span class="ip-label">IP</span>
        <span class="ip-value mono">{{ locationData.ip }}</span>
      </div>
      <div class="ip-item">
        <span class="ip-label">ISP</span>
        <span class="ip-value">{{ locationData.isp }}</span>
      </div>
      <div class="ip-item">
        <span class="ip-label">坐标</span>
        <span class="ip-value mono">{{ Number(locationData.lat)?.toFixed(4) || locationData.lat }}°, {{ Number(locationData.lon)?.toFixed(4) || locationData.lon }}°</span>
      </div>
      <div class="ip-item">
        <span class="ip-label">时区</span>
        <span class="ip-value">{{ locationData.timezone }}</span>
      </div>
    </div>
  </section>
</template>

<style scoped>
.ip-card {
  padding: 24px 28px;
  color: #fff;
  border-radius: 22px;
}

.ip-header {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 0.75rem;
  font-weight: 500;
  margin-bottom: 18px;
  color: rgba(255, 255, 255, 0.3);
  text-transform: uppercase;
  letter-spacing: 1.5px;
}

.ip-icon {
  width: 15px;
  height: 15px;
  opacity: 0.5;
}

.ip-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px 36px;
}

.ip-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.ip-label {
  font-size: 0.65rem;
  color: rgba(255, 255, 255, 0.2);
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: 500;
}

.ip-value {
  font-size: 0.88rem;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.6);
}

.ip-value.mono {
  font-family: 'SF Mono', 'Fira Code', monospace;
  letter-spacing: 0.5px;
  font-size: 0.82rem;
}

@media (max-width: 700px) {
  .ip-grid { grid-template-columns: 1fr; }
}
</style>
