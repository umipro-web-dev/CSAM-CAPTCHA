<template>
  <div class="app-container">
    <CaptchaWidget />
    <div class="credit">
      <a href="https://github.com/umipro-web-dev/CSAM-CAPTCHA" target="_blank" rel="noopener noreferrer">
        Created by umipro-web-dev
      </a>
    </div>
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
  position: relative;
}

.credit {
  position: fixed;
  bottom: 1.5rem;
  right: 1.5rem;
  font-size: clamp(10px, 1.2vw, 12px);
  color: #999;
}

.credit a {
  color: #999;
  text-decoration: none;
  transition: color 0.2s;
}

.credit a:hover {
  color: #1f8fe8;
  text-decoration: underline;
}
</style>
