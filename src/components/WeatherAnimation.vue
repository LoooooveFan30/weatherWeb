<template>
  <div class="weather-animation" :class="[weather, { night: isNight }]">
    <!-- 背景层 -->
    <div class="background-layer"></div>
    
    <!-- 太阳 -->
    <div class="celestial-body" v-if="showSun && !isNight">
      <div class="sun">
        <div class="sun-core"></div>
        <div class="sun-rays" v-if="showSunRays"></div>
        <div class="sun-glow"></div>
      </div>
    </div>

    <!-- 月亮 (夜间模式) -->
    <div class="celestial-body" v-if="isNight">
      <div class="moon">
        <div class="moon-body"></div>
        <div class="moon-glow"></div>
      </div>
    </div>

    <!-- 星星 (夜间晴天) -->
    <div class="stars" v-if="isNight && showSun">
      <div class="star" v-for="n in 30" :key="'star-' + n" :style="getStarStyle(n)"></div>
    </div>
    
    <!-- 云层 -->
    <div class="clouds" v-if="showClouds">
      <div class="cloud cloud-1"></div>
      <div class="cloud cloud-2"></div>
      <div class="cloud cloud-3" v-if="weather === 'overcast' || weather === 'rain' || weather === 'heavy_rain'"></div>
    </div>
    
    <!-- 雨滴 -->
    <div class="rain" v-if="showRain">
      <div class="drop" v-for="n in rainCount" :key="'rain-' + n" :style="getDropStyle(n)"></div>
    </div>
    
    <!-- 雪花 -->
    <div class="snow" v-if="showSnow">
      <div class="flake" v-for="n in snowCount" :key="'snow-' + n" :style="getFlakeStyle(n)"></div>
    </div>
    
    <!-- 闪电 -->
    <div class="lightning" v-if="weather === 'thunder'">
      <div class="bolt bolt-1"></div>
      <div class="bolt bolt-2"></div>
    </div>
    
    <!-- 雾气 -->
    <div class="fog" v-if="weather === 'fog' || weather === 'haze'">
      <div class="fog-layer fog-layer-1"></div>
      <div class="fog-layer fog-layer-2"></div>
      <div class="fog-layer fog-layer-3"></div>
    </div>
    
    <!-- 风 -->
    <div class="wind" v-if="weather === 'wind'">
      <div class="wind-line wind-line-1"></div>
      <div class="wind-line wind-line-2"></div>
      <div class="wind-line wind-line-3"></div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  weather: {
    type: String,
    default: 'clear'
  },
  // 是否为夜间模式
  isNight: {
    type: Boolean,
    default: false
  }
})

// 是否显示太阳（晴天、多云、风天时显示）
const showSun = computed(() => {
  return ['clear', 'cloudy', 'wind'].includes(props.weather)
})

// 是否显示云层
const showClouds = computed(() => {
  return ['cloudy', 'overcast', 'rain', 'heavy_rain', 'thunder', 'fog', 'haze'].includes(props.weather)
})

// 是否显示雨
const showRain = computed(() => {
  return ['rain', 'heavy_rain', 'thunder'].includes(props.weather)
})

// 是否显示雪
const showSnow = computed(() => {
  return props.weather === 'snow'
})

// 检测用户是否开启了减少动画偏好（减少粒子数量 + 降低动画速度）
const prefersReducedMotion = typeof window !== 'undefined'
  ? window.matchMedia('(prefers-reduced-motion: reduce)').matches
  : false

// 雨滴数量：开启减少动画时降低数量以提升性能
const rainCount = computed(() => {
  if (prefersReducedMotion) return 20
  return props.weather === 'heavy_rain' ? 80 : 40
})

// 雪花数量
const snowCount = computed(() => {
  return prefersReducedMotion ? 20 : 40
})

// 晴天时显示太阳光芒
const showSunRays = computed(() => {
  return props.weather === 'clear'
})

// 生成雨滴随机样式
function getDropStyle(n) {
  return {
    left: `${(n * 2.5) % 100}%`,
    animationDelay: `${(n * 0.13) % 2}s`,
    animationDuration: `${0.5 + (n % 5) * 0.1}s`
  }
}

// 生成雪花随机样式
function getFlakeStyle(n) {
  return {
    left: `${(n * 2.5) % 100}%`,
    animationDelay: `${(n * 0.23) % 3}s`,
    animationDuration: `${3 + (n % 4) * 0.5}s`,
    opacity: 0.6 + (n % 4) * 0.1
  }
}

// 生成星星随机位置
function getStarStyle(n) {
  return {
    left: `${(n * 3.33) % 100}%`,
    top: `${(n * 7.13 + 10) % 60}%`,
    animationDelay: `${(n * 0.37) % 4}s`,
    width: `${2 + (n % 3)}px`,
    height: `${2 + (n % 3)}px`
  }
}
</script>

<style scoped>
/* 减少动画偏好：降低动画速度 */
@media (prefers-reduced-motion: reduce) {
  .weather-animation * {
    animation-duration: 0.001s !important;
  }
}

.weather-animation {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  overflow: hidden;
  pointer-events: none;
}

/* 背景层 */
.background-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  transition: background 2s ease;
}

/* ===== 白天背景 ===== */
.clear .background-layer {
  background: linear-gradient(180deg, #4facfe 0%, #00f2fe 50%, #a8edea 100%);
}
.cloudy .background-layer {
  background: linear-gradient(180deg, #89b4d4 0%, #a8c8d8 50%, #c8dce8 100%);
}
.overcast .background-layer {
  background: linear-gradient(180deg, #6a7a8a 0%, #8a9aa8 50%, #a8b8c8 100%);
}
.rain .background-layer {
  background: linear-gradient(180deg, #5a6a7a 0%, #7a8a9a 50%, #9aa8b8 100%);
}
.heavy_rain .background-layer {
  background: linear-gradient(180deg, #4a5a6a 0%, #6a7a8a 50%, #8a9aa8 100%);
}
.snow .background-layer {
  background: linear-gradient(180deg, #e8f4f8 0%, #d0e8f0 50%, #b8dce8 100%);
}
.thunder .background-layer {
  background: linear-gradient(180deg, #2a3a4a 0%, #4a5a6a 50%, #6a7a8a 100%);
  animation: thunderFlash 3s infinite;
}
.fog .background-layer {
  background: linear-gradient(180deg, #c8d8e8 0%, #d8e8f0 50%, #e8f0f8 100%);
}
.haze .background-layer {
  background: linear-gradient(180deg, #d8c8a8 0%, #e8d8b8 50%, #f0e8c8 100%);
}
.wind .background-layer {
  background: linear-gradient(180deg, #a8c8d8 0%, #b8d8e8 50%, #c8e8f0 100%);
}

/* ===== 夜间背景 ===== */
.night.clear .background-layer {
  background: linear-gradient(180deg, #0a1628 0%, #162444 50%, #1f4068 100%);
}
.night.cloudy .background-layer {
  background: linear-gradient(180deg, #1a2a3a 0%, #2a3a4a 50%, #3a4a5a 100%);
}
.night.overcast .background-layer {
  background: linear-gradient(180deg, #151520 0%, #252535 50%, #353545 100%);
}
.night.rain .background-layer {
  background: linear-gradient(180deg, #101828 0%, #202838 50%, #303848 100%);
}
.night.snow .background-layer {
  background: linear-gradient(180deg, #1a2030 0%, #2a3040 50%, #3a4050 100%);
}
.night.thunder .background-layer {
  background: linear-gradient(180deg, #0a0a15 0%, #1a1a25 50%, #2a2a35 100%);
  animation: thunderFlash 3s infinite;
}
.night.fog .background-layer {
  background: linear-gradient(180deg, #1a2530 0%, #2a3540 50%, #3a4550 100%);
}

@keyframes thunderFlash {
  0%, 90%, 100% { opacity: 1; }
  92% { opacity: 0.7; }
  94% { opacity: 1; }
  96% { opacity: 0.8; }
}

/* ===== 太阳 ===== */
.celestial-body {
  position: absolute;
  top: 10%;
  right: 15%;
}

.sun {
  position: relative;
  width: 120px;
  height: 120px;
}

.sun-core {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 80px;
  height: 80px;
  background: radial-gradient(circle, #ffdd00 0%, #ffaa00 100%);
  border-radius: 50%;
  box-shadow: 0 0 60px #ffdd00, 0 0 100px #ffaa00;
  animation: sunPulse 4s ease-in-out infinite;
}

.sun-rays {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 120px;
  height: 120px;
  background: conic-gradient(from 0deg, transparent 0deg, rgba(255, 221, 0, 0.3) 10deg, transparent 20deg);
  border-radius: 50%;
  animation: sunRaysRotate 20s linear infinite;
}

.sun-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 140px;
  height: 140px;
  background: radial-gradient(circle, rgba(255, 221, 0, 0.2) 0%, transparent 70%);
  border-radius: 50%;
  animation: sunGlow 4s ease-in-out infinite;
}

@keyframes sunPulse {
  0%, 100% { transform: translate(-50%, -50%) scale(1); }
  50% { transform: translate(-50%, -50%) scale(1.05); }
}
@keyframes sunRaysRotate {
  from { transform: translate(-50%, -50%) rotate(0deg); }
  to { transform: translate(-50%, -50%) rotate(360deg); }
}
@keyframes sunGlow {
  0%, 100% { transform: translate(-50%, -50%) scale(1); opacity: 0.6; }
  50% { transform: translate(-50%, -50%) scale(1.2); opacity: 0.8; }
}

/* ===== 月亮 ===== */
.moon {
  position: relative;
  width: 100px;
  height: 100px;
}

.moon-body {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 70px;
  height: 70px;
  background: radial-gradient(circle at 35% 35%, #f5f5dc 0%, #ddd8b8 50%, #c8c090 100%);
  border-radius: 50%;
  box-shadow: 0 0 40px rgba(245, 245, 220, 0.4), 0 0 80px rgba(245, 245, 220, 0.2);
}

.moon-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 120px;
  height: 120px;
  background: radial-gradient(circle, rgba(245, 245, 220, 0.15) 0%, transparent 70%);
  border-radius: 50%;
}

/* ===== 星星 ===== */
.stars {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 60%;
}

.star {
  position: absolute;
  background: #fff;
  border-radius: 50%;
  animation: starTwinkle 3s ease-in-out infinite;
}

@keyframes starTwinkle {
  0%, 100% { opacity: 0.3; }
  50% { opacity: 1; }
}

/* ===== 云层 ===== */
.clouds {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.cloud {
  position: absolute;
  background: white;
  border-radius: 50%;
  filter: blur(1px);
  animation: cloudFloat linear infinite;
}

.night .cloud {
  background: rgba(180, 190, 200, 0.6);
}

.cloud::before,
.cloud::after {
  content: '';
  position: absolute;
  background: inherit;
  border-radius: 50%;
}

.cloud-1 {
  width: 200px;
  height: 60px;
  top: 15%;
  left: -200px;
  animation-duration: 40s;
  opacity: 0.9;
}
.cloud-1::before { width: 100px; height: 80px; top: -40px; left: 30px; }
.cloud-1::after { width: 80px; height: 60px; top: -30px; left: 80px; }

.cloud-2 {
  width: 150px;
  height: 45px;
  top: 25%;
  left: -150px;
  animation-duration: 50s;
  animation-delay: 10s;
  opacity: 0.8;
}
.cloud-2::before { width: 70px; height: 60px; top: -30px; left: 20px; }
.cloud-2::after { width: 60px; height: 45px; top: -25px; left: 60px; }

.cloud-3 {
  width: 180px;
  height: 55px;
  top: 35%;
  left: -180px;
  animation-duration: 45s;
  animation-delay: 20s;
  opacity: 0.7;
  background: #ddd;
}
.cloud-3::before { width: 90px; height: 70px; top: -35px; left: 25px; background: #ddd; }
.cloud-3::after { width: 70px; height: 55px; top: -30px; left: 75px; background: #ddd; }

@keyframes cloudFloat {
  from { transform: translateX(0); }
  to { transform: translateX(calc(100vw + 200px)); }
}

/* ===== 雨滴 ===== */
.rain {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.drop {
  position: absolute;
  top: -20px;
  width: 2px;
  height: 20px;
  background: linear-gradient(180deg, transparent 0%, rgba(174, 194, 224, 0.8) 50%, rgba(174, 194, 224, 0.5) 100%);
  animation: rainFall linear infinite;
}

@keyframes rainFall {
  0% { transform: translateY(-20px); opacity: 0; }
  10% { opacity: 1; }
  90% { opacity: 1; }
  95% { transform: translateY(100vh); opacity: 0.5; }
  100% { transform: translateY(100vh) scaleY(0.3); opacity: 0; }
}

/* ===== 雪花 ===== */
.snow {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.flake {
  position: absolute;
  top: -10px;
  width: 8px;
  height: 8px;
  background: white;
  border-radius: 50%;
  box-shadow: 0 0 5px rgba(255, 255, 255, 0.8);
  animation: snowFall linear infinite;
}

@keyframes snowFall {
  0% { transform: translateY(-10px) rotate(0deg) scale(1); opacity: 0; }
  10% { opacity: 1; }
  50% { transform: translateY(50vh) rotate(180deg) scale(0.8); }
  90% { opacity: 1; }
  100% { transform: translateY(100vh) rotate(360deg) scale(0.6); opacity: 0; }
}

/* ===== 闪电 ===== */
.lightning {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.bolt {
  position: absolute;
  width: 4px;
  height: 100px;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.9) 0%, rgba(200, 200, 255, 0.8) 50%, transparent 100%);
  animation: lightningFlash 3s infinite;
  clip-path: polygon(50% 0%, 60% 40%, 100% 40%, 60% 100%, 40% 60%, 0% 60%, 40% 0%);
}

.bolt-1 { top: 5%; left: 30%; animation-delay: 0s; }
.bolt-2 { top: 10%; left: 60%; animation-delay: 1.5s; height: 80px; }

@keyframes lightningFlash {
  0%, 89%, 91%, 93%, 100% { opacity: 0; }
  90%, 92% { opacity: 1; }
}

/* ===== 雾气 ===== */
.fog {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.fog-layer {
  position: absolute;
  width: 200%;
  height: 100%;
  background: linear-gradient(90deg, transparent 0%, rgba(255, 255, 255, 0.3) 25%, rgba(255, 255, 255, 0.5) 50%, rgba(255, 255, 255, 0.3) 75%, transparent 100%);
  animation: fogDrift linear infinite;
}

.night .fog-layer {
  background: linear-gradient(90deg, transparent 0%, rgba(180, 190, 200, 0.2) 25%, rgba(180, 190, 200, 0.35) 50%, rgba(180, 190, 200, 0.2) 75%, transparent 100%);
}

.fog-layer-1 { top: 20%; animation-duration: 30s; opacity: 0.6; }
.fog-layer-2 { top: 40%; animation-duration: 40s; animation-delay: 10s; opacity: 0.4; }
.fog-layer-3 { top: 60%; animation-duration: 35s; animation-delay: 20s; opacity: 0.5; }

@keyframes fogDrift {
  from { transform: translateX(-50%); }
  to { transform: translateX(0%); }
}

/* ===== 风 ===== */
.wind {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.wind-line {
  position: absolute;
  height: 2px;
  background: linear-gradient(90deg, transparent 0%, rgba(255, 255, 255, 0.6) 50%, transparent 100%);
  animation: windBlow linear infinite;
}

.wind-line-1 { top: 30%; width: 200px; animation-duration: 1.5s; }
.wind-line-2 { top: 50%; width: 150px; animation-duration: 2s; animation-delay: 0.5s; }
.wind-line-3 { top: 70%; width: 180px; animation-duration: 1.8s; animation-delay: 1s; }

@keyframes windBlow {
  0% { left: -200px; opacity: 0; }
  10% { opacity: 1; }
  90% { opacity: 1; }
  100% { left: 100%; opacity: 0; }
}

/* ===== 响应式 ===== */
@media (max-width: 768px) {
  .sun { width: 80px; height: 80px; }
  .sun-core { width: 60px; height: 60px; }
  .celestial-body { top: 5%; right: 10%; }
  .moon { width: 70px; height: 70px; }
  .moon-body { width: 50px; height: 50px; }
  .cloud-1 { width: 150px; height: 45px; }
  .cloud-2 { width: 120px; height: 36px; }
  .cloud-3 { width: 140px; height: 42px; }
}
</style>
