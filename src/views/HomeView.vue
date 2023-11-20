<script setup>
import { ref } from "vue";
import JSZip from 'jszip';
import CanvasImage from "../components/CanvasImage.vue";

const image = ref(null);
const canvasCompRefs = ref(null);
const sizeList = ref([32, 64, 128, 180, 200, 400]);

function handleFileSelect(ev) {
  const file = ev.target.files[0];
  ev.target.value = null;
  const img = new Image();
  img.onload = () => {
    image.value = img;
  };
  img.src = URL.createObjectURL(file);
}

function handleExport() {
  Promise.all(canvasCompRefs.value.map(item => {
    return item.toBlob().then(blob => {
      return {
        size: item.size,
        blob
      }
    });
  })).then(res => {
    const zip = new JSZip();
    const fIcons = zip.folder("icons");
    res.forEach(({size, blob}) => {
      fIcons.file(`icon-${size}.png`, blob);
    });
    zip.generateAsync({type:"blob"}).then(function(blob) {
      const url = URL.createObjectURL(blob);
      const downloadLink = document.createElement('a');
      downloadLink.href = url;
      downloadLink.download = 'icons.zip';
      downloadLink.click();
    });
  });
}

</script>

<template>
  <main class="view-home">
    <input id="imgInput" style="display: none;" type="file" @change="handleFileSelect" />

    <label class="btn-upload" for="imgInput">{{ image ? '重新选择' : '选择图片' }}</label>

    <div v-if="image">
      <CanvasImage
        v-for="size in sizeList"
        :key="size"
        :size="size"
        :image="image"
        ref="canvasCompRefs"
      />

      <button @click="handleExport">导出</button>
    </div>
  </main>
</template>

<style>
.view-home {
  padding: 16px;
}
.btn-upload {
  display: inline-block;
  cursor: pointer;
  padding: 8px 12px;
  background-color: coral;
  color: white;
  border-radius: 8px;
}
.btn-upload:active {
  background-color: rgba(255, 127, 80, .8);
}
</style>
