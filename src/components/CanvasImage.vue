<template>
  <div class="CanvasImage">
    <canvas ref="canvas" :width="size" :height="size"></canvas>
    <p>{{ size }}px</p>
  </div>
</template>

<script setup>
import {defineProps, ref, watchEffect, onMounted} from 'vue';

const props = defineProps(['size', 'image']);
let ctx = null;
const canvas = ref(null);

function drawImageOnCanvas(ctx, img) {
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

  console.log({iRatio, iw, ih, dx, dy, dw, dh});

  ctx.drawImage(img, 0, 0, iw, ih, dx, dy, dw, dh);
}

function toBlob() {
  return new Promise(resolve => {
    canvas.value.toBlob((blob) => {
      resolve(blob);
    }, 'image/png');
  });
}

defineExpose({
  toBlob,
  size: props.size
})

onMounted(() => {
  ctx = canvas.value.getContext('2d');
  watchEffect(() => {
    if (!props.size || !props.image) return;
    drawImageOnCanvas(ctx, props.image);
  });
})
</script>

<style scoped>
.CanvasImage {
  display: inline-block;
  margin: 8px;
}
.CanvasImage canvas {
  border: 1px solid #333;
}

.CanvasImage p {
  text-align: center;
}
</style>