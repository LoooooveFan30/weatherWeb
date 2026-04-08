<script setup>
import { ref } from 'vue'

const emit = defineEmits(['search'])
const query = ref('')

const doSearch = () => {
  const q = query.value.trim()
  if (!q) return
  emit('search', q)
}
</script>

<template>
  <div class="search-bar glass">
    <svg class="search-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
      <circle cx="11" cy="11" r="8"/>
      <path d="M21 21l-4.35-4.35"/>
    </svg>
    <input
      v-model="query"
      @keyup.enter="doSearch"
      type="text"
      placeholder="搜索城市天气..."
      class="search-input"
    />
    <button class="search-btn" @click="doSearch">
      <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M5 12h14M12 5l7 7-7 7"/>
      </svg>
    </button>
  </div>
</template>

<style scoped>
.search-bar {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 20px;
  color: #fff;
  border-radius: 18px;
  transition: all 0.3s ease;
}

.search-bar:focus-within {
  border-color: rgba(255, 255, 255, 0.2);
  box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.04);
}

.search-icon {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
  color: rgba(255, 255, 255, 0.3);
  transition: color 0.3s;
}

.search-bar:focus-within .search-icon {
  color: rgba(255, 255, 255, 0.5);
}

.search-input {
  flex: 1;
  background: transparent;
  border: none;
  outline: none;
  color: #fff;
  font-size: 0.9rem;
  font-family: inherit;
  font-weight: 400;
  letter-spacing: 0.2px;
}

.search-input::placeholder {
  color: rgba(255, 255, 255, 0.25);
  font-weight: 400;
}

.search-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  border: none;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.06);
  color: rgba(255, 255, 255, 0.5);
  cursor: pointer;
  transition: all 0.25s ease;
  flex-shrink: 0;
}

.search-btn:hover {
  background: rgba(255, 255, 255, 0.12);
  color: #fff;
  transform: translateX(2px);
}
</style>
