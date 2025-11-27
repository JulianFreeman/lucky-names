<script setup lang="ts">
import { ref, computed, onMounted, watch, reactive } from 'vue';
import LuckyWheel from './components/LuckyWheel.vue';
import RightPanel from './components/RightPanel.vue';

const namesInput = ref('');
const history = ref<string[]>([]);
const activeTab = ref<'names' | 'history'>('names');

const wheelSettings = reactive({
  fontSize: 1.0,
  textRadius: 0.6,
  textWidth: 0.8,
});

const isSettingsPanelOpen = ref(false);
const rightPanelRef = ref<HTMLElement | null>(null);
const settingsBtnRef = ref<HTMLElement | null>(null);

const toggleSettingsPanel = () => {
  isSettingsPanelOpen.value = !isSettingsPanelOpen.value;
};

const handleClickOutside = (event: MouseEvent) => {
  if (isSettingsPanelOpen.value) {
    const target = event.target as Node;
    const isClickInsidePanel = rightPanelRef.value?.contains(target);
    const isClickOnButton = settingsBtnRef.value?.contains(target);
    if (!isClickInsidePanel && !isClickOnButton) {
      isSettingsPanelOpen.value = false;
    }
  }
};

watch(isSettingsPanelOpen, (isOpen) => {
  if (isOpen) {
    document.addEventListener('mousedown', handleClickOutside);
  } else {
    document.removeEventListener('mousedown', handleClickOutside);
  }
});

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
  const savedSettings = localStorage.getItem('lucky-names-settings');
  if (savedSettings) {
    Object.assign(wheelSettings, JSON.parse(savedSettings));
  }
});

// Watch for changes and save to localStorage
watch(namesInput, (newValue) => {
  localStorage.setItem('lucky-names-input', newValue);
});
watch(history, (newValue) => {
  localStorage.setItem('lucky-names-history', JSON.stringify(newValue));
}, { deep: true });
watch(wheelSettings, (newValue) => {
  localStorage.setItem('lucky-names-settings', JSON.stringify(newValue));
});
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
    <div class="main-content" :class="{ 'shifted-left': isSettingsPanelOpen }">
      <button @click="toggleSettingsPanel" class="settings-btn" ref="settingsBtnRef">
        <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="currentColor"><path d="M0 0h24v24H0V0z" fill="none"/><path d="M12 8c1.1 0 2-.9 2-2s-.9-2-2-2-2 .9-2 2 .9 2 2 2zm0 2c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2zm0 6c-1.1 0-2 .9-2 2s.9 2 2 2 2-.9 2-2-.9-2-2-2z"/></svg>
      </button>
      <LuckyWheel 
        :prizes="names"
        :font-size-multiplier="wheelSettings.fontSize"
        :text-radius-factor="wheelSettings.textRadius"
        :text-width-multiplier="wheelSettings.textWidth"
        @winner-selected="handleWinnerSelected" 
      />
    </div>
    <Transition name="slide-fade">
      <div v-if="isSettingsPanelOpen" class="right-panel" ref="rightPanelRef">
        <RightPanel 
          v-model:font-size="wheelSettings.fontSize"
          v-model:text-radius="wheelSettings.textRadius"
          v-model:text-width="wheelSettings.textWidth"
        />
      </div>
    </Transition>
  </div>
</template>

<style scoped>
#app {
  display: flex;
  height: 100vh;
  background-color: var(--light-gray);
  color: var(--text-color);
  position: relative;
  overflow: hidden;
}

.sidebar {
  width: 320px;
  background-color: #ffffff;
  border-right: 1px solid var(--border-color);
  display: flex;
  flex-direction: column;
  box-shadow: none;
  padding: 0 16px;
  flex-shrink: 0;
  z-index: 10;
}

.main-content {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: var(--light-gray);
  padding: 40px;
  box-sizing: border-box;
  min-width: 0;
  position: relative;
  transition: padding-right 0.4s ease;
}

.main-content.shifted-left {
  padding-right: 320px; /* 280px panel width + 40px padding */
}

.settings-btn {
  position: absolute;
  top: 20px;
  right: 20px;
  z-index: 20;
  background-color: #fff;
  border: 1px solid var(--border-color);
  border-radius: 50%;
  width: 44px;
  height: 44px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  color: var(--dark-gray);
  transition: all 0.2s;
}
.settings-btn:hover {
  color: var(--primary-color);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.right-panel {
  position: absolute;
  top: 0;
  right: 0;
  width: 280px;
  height: 100%;
  background-color: #ffffff;
  border-left: 1px solid var(--border-color);
  padding: 20px;
  box-shadow: -4px 0 20px rgba(0,0,0,0.08);
  z-index: 100;
  box-sizing: border-box;
}

.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: transform 0.3s ease;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateX(100%);
}

.tabs {
  display: flex;
  padding-top: 8px;
}

.tabs button {
  flex: 1;
  padding: 10px 14px;
  border: none;
  background-color: transparent;
  cursor: pointer;
  font-size: 16px;
  font-weight: 500;
  color: var(--dark-gray);
  border-radius: 8px;
  transition: background-color 0.2s, color 0.2s;
}

.tabs button.active {
  background-color: var(--primary-color);
  color: #fff;
  font-weight: 500;
}

.tab-content {
  flex: 1;
  padding: 16px 0 20px;
  overflow-y: auto;
  min-height: 0;
}

.names-panel {
  height: 100%;
}

.names-panel textarea {
  width: 100%;
  height: 100%;
  border: none;
  background-color: var(--light-gray);
  border-radius: 8px;
  padding: 12px;
  font-size: 15px;
  font-family: 'Inter', sans-serif;
  resize: none;
  box-sizing: border-box;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.names-panel textarea:focus {
  outline: none;
  border-color: var(--primary-color);
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.2);
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
  padding: 12px 4px;
}

.history-panel h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 700;
  color: var(--text-color);
}

.clear-btn {
  border: none;
  background-color: var(--danger-color);
  color: white;
  padding: 6px 12px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: background-color 0.2s;
}

.clear-btn:hover {
  background-color: var(--danger-color-dark);
}

.history-panel ul {
  list-style: none;
  padding: 0;
  margin: 0;
  flex: 1;
  overflow-y: auto;
}

.history-panel li {
  padding: 10px 4px;
  border-bottom: 1px solid var(--border-color);
  color: var(--dark-gray);
  font-size: 15px;
}

.history-panel li:first-child {
  font-weight: 500;
  color: var(--text-color);
}

.no-history {
  color: var(--dark-gray);
  text-align: center;
  margin-top: 30px;
  font-size: 15px;
}
</style>
