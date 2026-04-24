<template>
  <div class="app-container">
    <CaptchaWidget />
  </div>
</template>

<script setup>
import CaptchaWidget from './components/CaptchaWidget.vue'
</script>

<style scoped>
.app-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
}
</style>
