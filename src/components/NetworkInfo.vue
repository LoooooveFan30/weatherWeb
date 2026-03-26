<script setup>
// 接收位置数据，展示 IP/ISP/经纬度/时区
defineProps({
  locationData: {
    type: Object,
    default: null
  }
})
</script>

<template>
  <!-- 网络信息卡片 -->
  <section class="ip-card glass" v-if="locationData">
    <div class="ip-header">
      <svg class="ip-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <rect x="2" y="3" width="20" height="14" rx="2"/>
        <path d="M8 21h8M12 17v4"/>
      </svg>
      <span>位置与网络信息</span>
    </div>
    <div class="ip-grid">
      <div class="ip-item">
        <span class="ip-label">IP 地址</span>
        <span class="ip-value mono">{{ locationData.ip }}</span>
      </div>
      <div class="ip-item">
        <span class="ip-label">ISP</span>
        <span class="ip-value">{{ locationData.isp }}</span>
      </div>
      <div class="ip-item">
        <span class="ip-label">经纬度</span>
        <span class="ip-value mono">{{ locationData.lat?.toFixed(4) }}°, {{ locationData.lon?.toFixed(4) }}°</span>
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
}

.ip-header {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 1rem;
  font-weight: 600;
  margin-bottom: 16px;
  color: rgba(255, 255, 255, 0.85);
}

.ip-icon {
  width: 20px;
  height: 20px;
}

.ip-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px 32px;
}

.ip-item {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.ip-label {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.5);
  text-transform: uppercase;
  letter-spacing: 1px;
}

.ip-value {
  font-size: 1rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.92);
  word-break: break-all;
}

.ip-value.mono {
  font-family: 'SF Mono', 'Fira Code', monospace;
  letter-spacing: 1px;
}

@media (max-width: 700px) {
  .ip-grid {
    grid-template-columns: 1fr;
  }
}
</style>
