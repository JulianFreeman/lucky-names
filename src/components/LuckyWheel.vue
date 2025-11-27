<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue';

const props = defineProps<{
    prizes: string[];
    fontSizeMultiplier: number;
    textRadiusFactor: number;
    textWidthMultiplier: number;
}>();

const emit = defineEmits<{
    (e: 'winner-selected', winner: string): void;
}>();

const wheelContainerRef = ref<HTMLDivElement | null>(null);
const canvasRef = ref<HTMLCanvasElement | null>(null);
const rotation = ref(0);
const isSpinning = ref(false);

const colors = ['#E66060', '#46B9B0', '#3EA5BC', '#E5C25C', '#8B50A4', '#29B866'];
const primaryColor = ref('#007bff'); // Default fallback

let resizeObserver: ResizeObserver | null = null;

const drawWheel = () => {
    const canvas = canvasRef.value;
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    if (!ctx) return;

    const arc = Math.PI * 2 / (props.prizes.length || 1);
    const radius = canvas.width / 2;

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Outer border
    ctx.strokeStyle = '#E0E0E0';
    ctx.lineWidth = 10;
    ctx.beginPath();
    ctx.arc(radius, radius, radius - 5, 0, Math.PI * 2);
    ctx.stroke();

    ctx.save();
    ctx.translate(radius, radius);
    ctx.rotate(rotation.value);

    const prizeCount = props.prizes.length || 1;
    // Base font size calculation
    const baseFontSizeDivisor = 15 + Math.max(0, prizeCount - 8) * 1.0;
    const baseFontSize = Math.max(12, Math.floor(radius / baseFontSizeDivisor));

    // Apply user-controlled multipliers
    const fontSize = baseFontSize * props.fontSizeMultiplier;
    const textRadius = radius * props.textRadiusFactor;

    props.prizes.forEach((prize, i) => {
        ctx.beginPath();
        ctx.fillStyle = colors[i % colors.length] ?? '#007bff';
        ctx.moveTo(0, 0);
        ctx.arc(0, 0, radius - 10, arc * i - arc / 2, arc * (i + 1) - arc / 2);
        ctx.closePath();
        ctx.fill();

        ctx.save();
        ctx.fillStyle = '#fff';
        ctx.font = `bold ${fontSize}px Inter`;
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.rotate(arc * i);

        // Truncate text if too long
        const maxTextWidth = (radius - textRadius) * props.textWidthMultiplier;
        let prizeText = prize;
        if (ctx.measureText(prizeText).width > maxTextWidth) {
            while (ctx.measureText(prizeText + '...').width > maxTextWidth && prizeText.length > 0) {
                prizeText = prizeText.slice(0, -1);
            }
            prizeText += '...';
        }
        ctx.fillText(prizeText, textRadius, 0);
        ctx.restore();
    });

    ctx.restore();

    // Center "SPIN" button
    const centerButtonRadius = radius / 4;
    ctx.fillStyle = '#fff';
    ctx.beginPath();
    ctx.arc(radius, radius, centerButtonRadius, 0, Math.PI * 2);
    ctx.closePath();
    ctx.fill();
    ctx.fillStyle = primaryColor.value;
    ctx.font = `bold ${Math.max(16, radius / 12)}px Inter`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText('SPIN', radius, radius);

    // Draw pointer on the right, pointing outwards
    const pointerSize = Math.max(15, radius / 25);
    const pointerWidth = Math.max(40, radius / 10);
    ctx.fillStyle = primaryColor.value;
    ctx.beginPath();
    ctx.moveTo(radius * 2 - pointerWidth, radius);
    ctx.lineTo(radius * 2, radius - pointerSize);
    ctx.lineTo(radius * 2, radius + pointerSize);
    ctx.closePath();
    ctx.fill();
};

const getRandomInt = (max: number) => {
    const randomBuffer = new Uint32Array(1);
    crypto.getRandomValues(randomBuffer);
    return (randomBuffer[0] ?? 0) % max;
};

const spin = () => {
    if (isSpinning.value || props.prizes.length === 0) return;
    isSpinning.value = true;

    const winnerIndex = getRandomInt(props.prizes.length);
    const totalRotations = 5;
    const arc = Math.PI * 2 / props.prizes.length;
    // Align winner to the right side (angle 0)
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
            emit('winner-selected', props.prizes[winnerIndex] ?? '');
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
    // Use CSS variables from root for consistency
    const rootStyle = getComputedStyle(document.documentElement);
    primaryColor.value = rootStyle.getPropertyValue('--primary-color').trim() || '#007bff';

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

watch(
    [() => props.fontSizeMultiplier, () => props.textRadiusFactor, () => props.textWidthMultiplier],
    () => {
        if (!isSpinning.value) {
            drawWheel();
        }
    }
);

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
    /* A more subtle shadow */
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.07);
}
</style>
