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
  padding: 20px;
  min-height: 100vh;
}
</style>
