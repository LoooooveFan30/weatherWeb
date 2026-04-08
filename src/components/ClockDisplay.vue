<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const time = ref('')
let timer = null

const updateTime = () => {
  const now = new Date()
  const h = String(now.getHours()).padStart(2, '0')
  const m = String(now.getMinutes()).padStart(2, '0')
  const s = String(now.getSeconds()).padStart(2, '0')
  time.value = `${h}:${m}:${s}`
}

onMounted(() => {
  updateTime()
  timer = setInterval(updateTime, 1000)
})

onUnmounted(() => {
  if (timer) clearInterval(timer)
})
</script>

<template>
  <div class="clock">{{ time }}</div>
</template>

<style scoped>
.clock {
  font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
  font-size: 1.05rem;
  font-weight: 500;
  letter-spacing: 1.5px;
  color: rgba(255, 255, 255, 0.45);
}
</style>
