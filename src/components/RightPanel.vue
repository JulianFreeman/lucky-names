<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
  fontSize: number;
  textRadius: number;
  textWidth: number;
}>();

const emit = defineEmits<{
  (e: 'update:fontSize', value: number): void;
  (e: 'update:textRadius', value: number): void;
  (e: 'update:textWidth', value: number): void;
}>();

const fontSizeModel = computed({
  get: () => props.fontSize,
  set: (value) => emit('update:fontSize', value),
});

const textRadiusModel = computed({
  get: () => props.textRadius,
  set: (value) => emit('update:textRadius', value),
});

const textWidthModel = computed({
  get: () => props.textWidth,
  set: (value) => emit('update:textWidth', value),
});
</script>

<template>
  <div class="controls-wrapper">
    <h4>外观设置</h4>
    <div class="control-group">
      <label for="font-size">字体大小</label>
      <input type="range" id="font-size" min="0.5" max="2.0" step="0.1" v-model.number="fontSizeModel" />
      <span>{{ fontSize.toFixed(1) }}</span>
    </div>
    <div class="control-group">
      <label for="text-radius">字体位置</label>
      <input type="range" id="text-radius" min="0.4" max="0.8" step="0.01" v-model.number="textRadiusModel" />
      <span>{{ textRadius.toFixed(2) }}</span>
    </div>
    <div class="control-group">
      <label for="text-width">字体宽度</label>
      <input type="range" id="text-width" min="0.4" max="1.0" step="0.05" v-model.number="textWidthModel" />
      <span>{{ textWidth.toFixed(2) }}</span>
    </div>
  </div>
</template>

<style scoped>
.controls-wrapper h4 {
  margin-top: 0;
  margin-bottom: 24px;
  font-weight: 700;
  color: var(--text-color);
}

.control-group {
  margin-bottom: 20px;
}

.control-group label {
  display: block;
  font-size: 14px;
  font-weight: 500;
  margin-bottom: 8px;
  color: var(--dark-gray);
}

.control-group input[type="range"] {
  -webkit-appearance: none;
  appearance: none;
  width: calc(100% - 40px);
  margin-right: 10px;
  background: var(--medium-gray);
  height: 8px;
  border-radius: 4px;
  outline: none;
  transition: opacity 0.2s;
}

.control-group input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: var(--primary-color);
  cursor: pointer;
}

.control-group input[type="range"]::-moz-range-thumb {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: var(--primary-color);
  cursor: pointer;
}

.control-group span {
  display: inline-block;
  width: 30px;
  text-align: right;
  font-size: 14px;
  color: var(--text-color);
}
</style>
