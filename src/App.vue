<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue';
import LuckyWheel from './components/LuckyWheel.vue';

const namesInput = ref('');
const history = ref<string[]>([]);
const activeTab = ref<'names' | 'history'>('names');

const names = computed(() => {
  return namesInput.value.split('\n').filter(name => name.trim() !== '').map(name => name.trim());
});

const handleWinnerSelected = (winner: string) => {
  history.value.unshift(winner);
};

const clearHistory = () => {
  if (window.confirm('确定要清空所有历史记录吗？')) {
    history.value = [];
  }
};

// Load data from localStorage on component mount
onMounted(() => {
  const savedNames = localStorage.getItem('lucky-names-input');
  if (savedNames) {
    namesInput.value = savedNames;
  }
  const savedHistory = localStorage.getItem('lucky-names-history');
  if (savedHistory) {
    history.value = JSON.parse(savedHistory);
  }
});

// Watch for changes and save to localStorage
watch(namesInput, (newValue) => {
  localStorage.setItem('lucky-names-input', newValue);
});

watch(history, (newValue) => {
  localStorage.setItem('lucky-names-history', JSON.stringify(newValue));
}, { deep: true });
</script>

<template>
  <div id="app">
    <div class="sidebar">
      <div class="tabs">
        <button @click="activeTab = 'names'" :class="{ active: activeTab === 'names' }">名单</button>
        <button @click="activeTab = 'history'" :class="{ active: activeTab === 'history' }">历史</button>
      </div>
      <div class="tab-content">
        <div v-if="activeTab === 'names'" class="names-panel">
          <textarea v-model="namesInput" placeholder="请输入参与抽奖的名单，每行一个"></textarea>
        </div>
        <div v-if="activeTab === 'history'" class="history-panel">
          <div class="history-header">
            <h3>历史记录</h3>
            <button @click="clearHistory" class="clear-btn" v-if="history.length > 0">清空记录</button>
          </div>
          <ul v-if="history.length > 0">
            <li v-for="(winner, index) in history" :key="index">{{ winner }}</li>
          </ul>
          <p v-else class="no-history">暂无历史记录</p>
        </div>
      </div>
    </div>
    <div class="main-content">
      <LuckyWheel :prizes="names" @winner-selected="handleWinnerSelected" />
    </div>
  </div>
</template>

<style scoped>
#app {
  display: flex;
  height: 100vh;
  background-color: #fff;
}

.sidebar {
  width: 300px;
  background-color: #fff;
  border-right: 1px solid #e0e0e0;
  display: flex;
  flex-direction: column;
  box-shadow: 2px 0 5px rgba(0,0,0,0.05);
}

.tabs {
  display: flex;
  border-bottom: 1px solid #e0e0e0;
}

.tabs button {
  flex: 1;
  padding: 12px;
  border: none;
  background-color: #f7f7f7;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
  color: #555;
}

.tabs button.active {
  background-color: #fff;
  font-weight: bold;
  color: #000;
}

.tabs button:not(.active):hover {
  background-color: #efefef;
}

.tab-content {
  flex: 1;
  padding: 15px;
  overflow-y: auto;
  min-height: 0; /* Prevents flexbox overflow issue */
}

/* Modern scrollbar for webkit browsers */
.tab-content::-webkit-scrollbar {
  width: 6px;
}
.tab-content::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 3px;
}
.tab-content::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}

.names-panel {
  height: 100%;
}

.names-panel textarea {
  width: 100%;
  height: 100%;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 10px;
  font-size: 14px;
  resize: none;
  box-sizing: border-box;
}

.history-panel {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.history-panel h3 {
  margin: 0;
  font-size: 18px;
  color: #333;
}

.clear-btn {
  border: none;
  background-color: #ff4d4f;
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  transition: background-color 0.3s;
}

.clear-btn:hover {
  background-color: #d9363e;
}

.history-panel ul {
  list-style: none;
  padding: 0;
  margin: 0;
  flex: 1;
  overflow-y: auto;
}

.history-panel li {
  padding: 8px 0;
  border-bottom: 1px solid #eee;
  color: #555;
}

.no-history {
  color: #999;
  text-align: center;
  margin-top: 20px;
}

.main-content {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f2f5; /* Different background for contrast */
  padding: 40px;
  box-sizing: border-box;
}
</style>
