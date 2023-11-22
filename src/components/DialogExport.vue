<!-- eslint-disable vue/no-mutating-props -->
<template>
  <div v-if="params" class="dialog dialog-export" @click="handleClick">
    <div class="dialog-inner">
      <div class="dialog-content">
        <div
          class="img"
          v-for="(item, i) in imgSrcs"
          :key="i"
        >
          <img :src="item.src" :alt="item.size + 'px'">
          <p>{{ item.size }}px</p>
        </div>
      </div>
      <button class="btn-download" type="button" @click="downloadZip">下载</button>
    </div>
  </div>
</template>

<script>
  import JSZip from 'jszip';

  export default {
    props: {
      params: Array
    },
    computed: {
      imgSrcs() {
        if (!this.params?.length) return [];
        return this.params.map(({size, blob}) => ({
          size, src: URL.createObjectURL(blob)
        }));
      }
    },
    methods: {
      handleClick(ev) {
        if (ev.target === ev.currentTarget) {
          this.$emit('update:params', null);
        }
      },
      downloadZip() {
        const zip = new JSZip();
        const fIcons = zip.folder("icons");
        this.params.forEach(({size, blob}) => {
          fIcons.file(`icon-${size}.png`, blob);
        });
        zip.generateAsync({type:"blob"}).then(function(blob) {
          const url = URL.createObjectURL(blob);
          const downloadLink = document.createElement('a');
          downloadLink.href = url;
          downloadLink.download = 'icons.zip';
          downloadLink.click();
        });
      }
    }
  }
</script>

<style scoped>
.dialog {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, .2);
}

.dialog-inner {
  width: 50%;
  background-color: white;
  padding: 16px;
  border-radius: 12px;
  position: relative;
}
.dialog-content {
  min-height: 50vh;
  max-height: 80vh;
  overflow: auto;
}

.btn-download {
  position: absolute;
  right: 16px;
  bottom: 32px;
}
</style>