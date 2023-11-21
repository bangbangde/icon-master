<script setup>
import { ref, reactive } from "vue";
import JSZip from 'jszip';
import OptionsForm from '../components/OptionsForm.vue';
import CanvasImage from "../components/CanvasImage.vue";
import { toPng } from '../utils/svg2png';

const image = ref(null);
const canvasCompRefs = ref(null);
const sizeList = ref([32, 64, 128, 180, 200, 400]);
const options = reactive({
  width: null,
  height: null
});

async function handleFileSelect(ev) {
  const file = ev.target.files[0];
  ev.target.value = null;
  console.log(file);
  if (!file.type.startsWith('image')) {
    alert('你选择的不是图片！');
    return;
  }
  if (file.type === `image/svg+xml`) {
    image.value = await toPng({
      width: 500,
      height: 500,
      svg: URL.createObjectURL(file)
    });
  } else {
    await new Promise(resolve => {
      const img = new Image();
      img.onload = () => {
        image.value = img;
        resolve();
      };
      img.src = URL.createObjectURL(file);
    })
  }

  options.width = image.value.width;
  options.height = image.value.height;
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
    <CanvasImage
      class="canvas"
      :size="size"
      :image="image"
      ref="canvasCompRefs"
    />
    <aside>
      <input id="imgInput" style="display: none;" type="file" @change="handleFileSelect" />
      <label class="btn-upload" for="imgInput">{{ image ? '重新选择' : '选择图片' }}</label>
      <OptionsForm
        :options="options"
      />
    </aside>
  </main>
</template>

<style>
.view-home {
  padding: 16px;
  display: flex;
}
.canvas {
  width: 400px;
  height: 400px;
  background: 
    0 0/200px linear-gradient(45deg, #000 25%, transparent 0, transparent 75%, #000 0),
    100px 100px/ 200px linear-gradient(45deg, #c80909 25%, transparent 0, transparent 75%, #c80909 0);
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
