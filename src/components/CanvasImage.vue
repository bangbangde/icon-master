<template>
  <div class="CanvasImage">
    <canvas ref="refCanvas" :width="size" :height="size"></canvas>
    <p>{{ info }}</p>
  </div>
</template>

<script setup>
import {defineProps, ref, watchEffect, onMounted, computed} from 'vue';

function clipRadius (ctx, x, y, w, h, r) {
    const min_size = Math.min(w, h);
    if (r > min_size / 2) r = min_size / 2;
    ctx.beginPath();
    ctx.moveTo(x + r, y);
    ctx.arcTo(x + w, y, x + w, y + h, r);
    ctx.arcTo(x + w, y + h, x, y + h, r);
    ctx.arcTo(x, y + h, x, y, r);
    ctx.arcTo(x, y, x + w, y, r);
    ctx.closePath();
    ctx.clip();
}

const props = defineProps(['size', 'image', 'radius']);
let ctx = null;
const refCanvas = ref(null);

function drawImageOnCanvas(ctx, img, r) {
  if (!ctx || !img) return;
  ctx.save();
  const { width: cw, height: ch } = ctx.canvas;
  const { width: iw, height: ih } = img;
  let dx = 0, 
      dy = 0, 
      dw, 
      dh, 
      cRatio = cw / ch, 
      iRatio = iw / ih;
  ctx.clearRect(0, 0, cw, ch);

  if (cRatio > iRatio) {
    dh = ch;
    dw = dh * iRatio;
    dx = (cw - dw) / 2;
  } else {
    dw = cw;
    dh = dw / iRatio;
    dy = (ch - dh) / 2;
  }
  if (r) {
    clipRadius(ctx, dx, dy, dw, dh, r);
  }
  ctx.drawImage(img, 0, 0, iw, ih, dx, dy, dw, dh);
  ctx.restore();
}

async function toBlob(size) {
  let canvas;
  if (size) {
    canvas = document.createElement('canvas');
    canvas.width = canvas.height = size;
    const ctx = canvas.getContext('2d');
    drawImageOnCanvas(ctx, props.image, props.radius);
  } else {
    size = props.size;
    canvas = refCanvas.value;
  }
  
  return new Promise(resolve => {
    canvas.toBlob((blob) => {
      resolve({ size, blob });
    }, 'image/png');
  });
}

const info = computed(() => {
  if (!props.image) return '';
  return `width: ${props.image.width} height: ${props.image.height}`;
})

onMounted(() => {
  ctx = refCanvas.value.getContext('2d');
  watchEffect(() => {
    if (!props.image || !props.size) return;
    drawImageOnCanvas(ctx, props.image, props.radius);
  });
});

defineExpose({
  toBlob
});
</script>

<style scoped>
.CanvasImage {
  display: block;
}
.CanvasImage canvas {
  --color: #e3e3e3;
  background: 
    0 0/40px 40px linear-gradient(45deg, var(--color) 25%, transparent 0, transparent 75%, var(--color) 0),
    20px 20px/40px 40px linear-gradient(45deg, var(--color) 25%, transparent 0, transparent 75%, var(--color) 0);
}

.CanvasImage p {
  text-align: center;
}
</style>