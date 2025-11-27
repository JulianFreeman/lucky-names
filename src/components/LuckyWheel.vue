<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue';

const props = defineProps<{
  prizes: string[];
}>();

const emit = defineEmits<{
  (e: 'winner-selected', winner: string): void;
}>();

const wheelContainerRef = ref<HTMLDivElement | null>(null);
const canvasRef = ref<HTMLCanvasElement | null>(null);
const rotation = ref(0);
const isSpinning = ref(false);

const colors = ['#FFD700', '#FF6347', '#ADFF2F', '#40E0D0', '#EE82EE', '#FF69B4'];

let resizeObserver: ResizeObserver | null = null;

const drawWheel = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  const arc = Math.PI * 2 / (props.prizes.length || 1);
  const radius = canvas.width / 2;

  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.save();
  ctx.translate(radius, radius);
  ctx.rotate(rotation.value);

  const textRadius = radius * 0.65;
  const fontSize = Math.max(12, Math.floor(radius / 20));

  props.prizes.forEach((prize, i) => {
    ctx.beginPath();
    ctx.fillStyle = colors[i % colors.length];
    ctx.moveTo(0, 0);
    ctx.arc(0, 0, radius - 5, arc * i - arc / 2, arc * (i + 1) - arc / 2);
    ctx.closePath();
    ctx.fill();

    ctx.save();
    ctx.fillStyle = '#000';
    ctx.font = `bold ${fontSize}px Arial`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.rotate(arc * i);
    ctx.fillText(prize, textRadius, 0);
    ctx.restore();
  });

  ctx.restore();

  // Draw pointer on the right
  const pointerSize = Math.max(15, radius / 25);
  const pointerWidth = Math.max(40, radius / 10);
  ctx.fillStyle = '#333';
  ctx.beginPath();
  ctx.moveTo(radius * 2 - 5, radius);
  ctx.lineTo(radius * 2 - pointerWidth, radius - pointerSize);
  ctx.lineTo(radius * 2 - pointerWidth, radius + pointerSize);
  ctx.closePath();
  ctx.fill();
};

const getRandomInt = (max: number) => {
  const randomBuffer = new Uint32Array(1);
  crypto.getRandomValues(randomBuffer);
  return randomBuffer[0] % max;
};

const spin = () => {
  if (isSpinning.value || props.prizes.length === 0) return;
  isSpinning.value = true;

  const winnerIndex = getRandomInt(props.prizes.length);
  const totalRotations = 5;
  const arc = Math.PI * 2 / props.prizes.length;
  // Correct the target rotation to align the center of the prize with the pointer at 0 radians (right side)
  const targetRotation = (totalRotations * Math.PI * 2) - (winnerIndex * arc) + (Math.random() * arc * 0.8 - arc * 0.4);

  let start: number | null = null;
  const duration = 5000; // 5 seconds
  const startRotation = rotation.value;

  const animate = (timestamp: number) => {
    if (!start) start = timestamp;
    const progress = timestamp - start;
    const easedProgress = 1 - Math.pow(1 - (progress / duration), 4); // easeOutQuart

    rotation.value = startRotation + (targetRotation - startRotation) * easedProgress;

    drawWheel();

    if (progress < duration) {
      requestAnimationFrame(animate);
    } else {
      rotation.value = targetRotation % (Math.PI * 2);
      drawWheel();
      isSpinning.value = false;
      emit('winner-selected', props.prizes[winnerIndex]);
    }
  };

  requestAnimationFrame(animate);
};

const resizeCanvas = () => {
  const container = wheelContainerRef.value;
  const canvas = canvasRef.value;
  if (container && canvas) {
    const size = Math.min(container.clientWidth, container.clientHeight);
    canvas.width = size;
    canvas.height = size;
    drawWheel();
  }
};

onMounted(() => {
  if (wheelContainerRef.value) {
    resizeObserver = new ResizeObserver(resizeCanvas);
    resizeObserver.observe(wheelContainerRef.value);
  }
  resizeCanvas();
});

onUnmounted(() => {
  if (resizeObserver && wheelContainerRef.value) {
    resizeObserver.unobserve(wheelContainerRef.value);
  }
});

watch(() => props.prizes, () => {
  if (!isSpinning.value) {
    rotation.value = 0;
    drawWheel();
  }
}, { deep: true });

</script>

<template>
  <div ref="wheelContainerRef" class="wheel-container">
    <canvas ref="canvasRef" @click="spin"></canvas>
  </div>
</template>

<style scoped>
.wheel-container {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

canvas {
  cursor: pointer;
  border-radius: 50%;
  box-shadow: 0 0 25px rgba(0,0,0,0.15);
}
</style>
