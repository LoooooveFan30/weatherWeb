<script setup>
import { ref } from 'vue'

const emit = defineEmits(['search'])

// 搜索关键词
const query = ref('')

// 点击搜索或回车时，将关键词发送给父组件处理
const doSearch = () => {
  const q = query.value.trim()
  if (!q) return
  emit('search', q)
}
</script>

<template>
  <div class="search-bar glass">
    <svg class="search-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <circle cx="11" cy="11" r="8"/>
      <path d="M21 21l-4.35-4.35"/>
    </svg>
    <input
      v-model="query"
      @keyup.enter="doSearch"
      type="text"
      placeholder="搜索城市..."
      class="search-input"
    />
    <button class="search-btn" @click="doSearch">搜索</button>
  </div>
</template>

<style scoped>
.search-bar {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 16px;
  color: #fff;
}

.search-icon {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
  color: rgba(255, 255, 255, 0.6);
}

.search-input {
  flex: 1;
  background: transparent;
  border: none;
  outline: none;
  color: #fff;
  font-size: 0.95rem;
  font-family: inherit;
}

.search-input::placeholder {
  color: rgba(255, 255, 255, 0.4);
}

.search-btn {
  padding: 6px 16px;
  border: none;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.18);
  color: #fff;
  font-size: 0.85rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
  white-space: nowrap;
}

.search-btn:hover {
  background: rgba(255, 255, 255, 0.3);
}
</style>
