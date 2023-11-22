<script setup>
import { RouterView } from 'vue-router'
</script>

<template>
  <div class="app">
    <header class="app-head">
      <nav>
        <div class="page-title">Icon Master</div>
      </nav>
    </header>

    <RouterView class="router-view" />
  </div>
</template>

<style scoped>
.app {
  height: 100vh;
  display: flex;
  flex-direction: column;
}
.app-head {
  flex: 0 0 auto;
}
.router-view {
  flex: 1 1 auto;
  height: 0;
}
</style>
