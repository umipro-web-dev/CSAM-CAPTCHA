<template>
  <div class="captcha-widget">
    <div class="captcha-container">
      <div class="captcha-box">
        <!-- Main Section -->
        <div v-if="!showChallenge && !verified" class="main-section">
          <div class="checkbox-large">
            <div v-if="!loading" class="checkbox-wrapper">
              <input 
                type="checkbox" 
                id="captcha-checkbox"
                v-model="isChecked"
                @change="handleCheck"
              >
            </div>
            <div v-else class="checkbox-spinner">
              <div class="spinner-checkbox"></div>
            </div>
          </div>
          <div class="main-text-area">
            <label for="captcha-checkbox" class="main-label">
              普通科ではありません
            </label>
            <p class="sub-text">H-CAPTCHA</p>
          </div>
          <div class="logo-area">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 120 120" class="logo-img" aria-hidden="true" focusable="false">
              <defs>
                <linearGradient id="csGradient" x1="0%" y1="0%" x2="100%" y2="100%">
                  <stop offset="0%" style="stop-color:#1f8fe8;stop-opacity:1" />
                  <stop offset="100%" style="stop-color:#0052a3;stop-opacity:1" />
                </linearGradient>
                <filter id="shadow" x="-50%" y="-50%" width="200%" height="200%">
                  <feDropShadow dx="0" dy="3" stdDeviation="4" flood-opacity="0.15"/>
                </filter>
              </defs>
              
              <!-- Background with subtle pattern -->
              <rect width="120" height="120" fill="#f8fbff" filter="url(#shadow)"/>
              
              <!-- Corner accent circles -->
              <circle cx="15" cy="15" r="8" fill="url(#csGradient)" opacity="0.35"/>
              <circle cx="105" cy="105" r="8" fill="url(#csGradient)" opacity="0.35"/>
              <circle cx="105" cy="15" r="5" fill="url(#csGradient)" opacity="0.25"/>
              <circle cx="15" cy="105" r="5" fill="url(#csGradient)" opacity="0.25"/>
              
              <!-- Decorative lines -->
              <line x1="20" y1="35" x2="40" y2="35" stroke="url(#csGradient)" stroke-width="2" stroke-linecap="round" opacity="0.6"/>
              <line x1="80" y1="35" x2="100" y2="35" stroke="url(#csGradient)" stroke-width="2" stroke-linecap="round" opacity="0.6"/>
              <line x1="20" y1="90" x2="40" y2="90" stroke="url(#csGradient)" stroke-width="2" stroke-linecap="round" opacity="0.6"/>
              <line x1="80" y1="90" x2="100" y2="90" stroke="url(#csGradient)" stroke-width="2" stroke-linecap="round" opacity="0.6"/>
              
              <!-- Dots decoration -->
              <circle cx="50" cy="28" r="2" fill="url(#csGradient)" opacity="0.7"/>
              <circle cx="70" cy="28" r="2" fill="url(#csGradient)" opacity="0.7"/>
              <circle cx="50" cy="97" r="2" fill="url(#csGradient)" opacity="0.7"/>
              <circle cx="70" cy="97" r="2" fill="url(#csGradient)" opacity="0.7"/>
              
              <!-- Main text - CSAM -->
              <text x="60" y="75" font-size="30" font-weight="800" text-anchor="middle" fill="url(#csGradient)" font-family="'Segoe UI', -apple-system, sans-serif" letter-spacing="1">CSAM</text>
              
              <!-- Bottom accent bar -->
              <rect x="30" y="85" width="60" height="3" fill="url(#csGradient)" opacity="0.8" rx="1.5"/>
            </svg>
          </div>
        </div>

        <!-- Loading Section -->
        <div v-else-if="loading && !verified" class="loading-section">
          <div class="spinner"></div>
          <p>検証中...</p>
        </div>

        <!-- Challenge Section -->
        <div v-else-if="showChallenge && !verified" class="challenge-section">
          <div class="challenge-header">
            <h3>{{ currentChallenge.prompt }}</h3>
            <button class="close-btn" @click="resetCaptcha">✕</button>
          </div>

          <div class="challenge-instruction">
            <p>{{ currentChallenge.instruction }}</p>
          </div>

          <div class="image-grid-large">
            <div
              v-for="(image, index) in currentChallenge.images"
              :key="index"
              class="image-item-large"
              :class="{ selected: selectedImages.includes(index) }"
              @click="toggleImage(index)"
            >
              <div class="image-display">
                <img :src="image.src" :alt="image.alt" class="image-file" />
              </div>
            </div>
          </div>

          <div class="challenge-actions">
            <button class="btn-skip" @click="skipChallenge">スキップ</button>
            <button class="btn-verify" @click="verifyChallenge">確認</button>
          </div>

          <div v-if="error" class="error-message">
            ⚠️ {{ error }}
          </div>
        </div>

        <!-- Success Section -->
        <div v-else-if="verified" class="success-section">
          <div class="success-icon">✓</div>
          <div class="success-text">
            <p class="success-title">おめでとうございます！</p>
            <p class="success-sub">あなたは、理数科もしくは理数科と同等の学力を有していると認定されました！</p>
          </div>
        </div>

        <div class="captcha-footer"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['verified'])

const isChecked = ref(false)
const showChallenge = ref(false)
const verified = ref(false)
const loading = ref(false)
const selectedImages = ref([])
const error = ref('')

// 画像ファイル情報を定義
const correctImages = [
  'a.png', 'b.png', 'c.png', 'd.png', 'e.png', 'f.png', 'g.png', 'h.png'
]

const wrongImages = [
  'a.png', 'b.png', 'c.png', 'd.png', 'e.png', 'f.png', 'g.png', 'h.png',
  'i.png', 'j.png', 'k.png', 'l.png', 'm.png', 'n.png', 'o.png', 'p.png',
  'q.png', 'r.png', 's.png', 't.png'
]

const generateChallenge = () => {
  // 正解の数をランダムに生成（1～3）
  const correctCount = Math.floor(Math.random() * 3) + 1
  
  // 正解画像をランダムに選択（correctImgsから）
  const selectedCorrectImages = []
  const tempCorrectImages = [...correctImages]
  for (let i = 0; i < correctCount; i++) {
    const randomIdx = Math.floor(Math.random() * tempCorrectImages.length)
    selectedCorrectImages.push(tempCorrectImages[randomIdx])
    tempCorrectImages.splice(randomIdx, 1)
  }
  
  // 不正解画像を選択（wrongImgsから9-correctCount個）
  const selectedWrongImages = []
  const tempWrongImages = [...wrongImages]
  const wrongCount = 9 - correctCount
  
  for (let i = 0; i < wrongCount; i++) {
    const randomIdx = Math.floor(Math.random() * tempWrongImages.length)
    selectedWrongImages.push(tempWrongImages[randomIdx])
    tempWrongImages.splice(randomIdx, 1)
  }
  
  // 正解と不正解を組み合わせ
  const images = [
    ...selectedCorrectImages.map(filename => ({ filename, isCorrect: true })),
    ...selectedWrongImages.map(filename => ({ filename, isCorrect: false }))
  ]
  
  // シャッフル
  images.sort(() => Math.random() - 0.5)
  
  // 正解のインデックスを取得
  const correctIndices = images
    .map((img, idx) => img.isCorrect ? idx : -1)
    .filter(idx => idx !== -1)
  
  return {
    prompt: '1はどれか？',
    instruction: '数字の1を含む画像をすべてクリックしてください',
    images: images.map(img => ({
      src: `/images/${img.isCorrect ? 'correctImgs' : 'wrongImgs'}/${img.filename}`,
      alt: img.isCorrect ? 'correct' : 'wrong'
    })),
    correctIndices
  }
}

const currentChallenge = ref(generateChallenge())

const handleCheck = () => {
  if (isChecked.value) {
    loading.value = true
    setTimeout(() => {
      loading.value = false
      showChallenge.value = true
      currentChallenge.value = generateChallenge()
      selectedImages.value = []
    }, 1200)
  } else {
    resetCaptcha()
  }
}

const toggleImage = (index) => {
  const idx = selectedImages.value.indexOf(index)
  if (idx > -1) {
    selectedImages.value.splice(idx, 1)
  } else {
    selectedImages.value.push(index)
  }
}

const verifyChallenge = () => {
  const correctIndices = currentChallenge.value.correctIndices.sort((a, b) => a - b)
  const selected = selectedImages.value.sort((a, b) => a - b)
  
  const isCorrect = 
    correctIndices.length === selected.length &&
    correctIndices.every((val, idx) => val === selected[idx])
  
  if (!isCorrect) {
    error.value = '選択が正しくありません。もう一度試してください。'
    setTimeout(() => {
      error.value = ''
    }, 3000)
    return
  }
  
  verified.value = true
  emit('verified', true)
  setTimeout(() => {
    showChallenge.value = false
  }, 1500)
}

const skipChallenge = () => {
  // 新しいチャレンジを生成して切り替える
  currentChallenge.value = generateChallenge()
  selectedImages.value = []
  error.value = ''
}

const resetCaptcha = () => {
  isChecked.value = false
  showChallenge.value = false
  verified.value = false
  loading.value = false
  selectedImages.value = []
  emit('verified', false)
}
</script>

<style scoped>
.captcha-widget {
  margin: 0;
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.captcha-container {
  width: 100%;
  max-width: 600px;
}

.captcha-box {
  background: white;
  border: 1px solid #9e9e9e;
  border-radius: 2px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.12);
}

/* Main Section */
.main-section {
  display: flex;
  align-items: center;
  gap: 30px;
  padding: 40px 50px;
  min-height: 180px;
}

.checkbox-large {
  flex-shrink: 0;
}

.checkbox-wrapper {
  width: 48px;
  height: 48px;
}

.checkbox-large input[type="checkbox"] {
  width: 48px;
  height: 48px;
  cursor: pointer;
  accent-color: #1f8fe8;
}

.checkbox-spinner {
  width: 48px;
  height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.spinner-checkbox {
  width: 48px;
  height: 48px;
  border: 4px solid #e8e8e8;
  border-top-color: #1f8fe8;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.main-text-area {
  flex: 1;
}

.main-label {
  display: block;
  font-size: 28px;
  font-weight: 500;
  color: #333;
  margin-bottom: 6px;
  cursor: pointer;
  user-select: none;
}

.sub-text {
  margin: 0;
  font-size: 15px;
  color: #999;
}

.logo-area {
  flex-shrink: 0;
  width: 80px;
  height: 80px;
}

.logo-img {
  width: 100%;
  height: 100%;
}

/* Loading Section */
.loading-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 15px;
  padding: 40px;
  min-height: 200px;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f0f0f0;
  border-top-color: #1f8fe8;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

.loading-section p {
  margin: 0;
  font-size: 15px;
  color: #666;
}

/* Challenge Section */
.challenge-section {
  padding: 40px;
}

.challenge-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
}

.challenge-header h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: #333;
}

.close-btn {
  background: none;
  border: none;
  font-size: 20px;
  color: #999;
  cursor: pointer;
  padding: 4px 8px;
  transition: color 0.2s;
}

.close-btn:hover {
  color: #333;
}

.challenge-instruction {
  text-align: center;
  margin-bottom: 25px;
}

.challenge-instruction p {
  margin: 0;
  font-size: 14px;
  color: #666;
}

.image-grid-large {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-bottom: 30px;
}

.image-item-large {
  aspect-ratio: 1;
  border: 4px solid #d3d3d3;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  background: #fafafa;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  position: relative;
}

.image-item-large:hover {
  border-color: #999;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.image-item-large.selected {
  border-color: #1f8fe8;
  background: #e8f4fb;
  box-shadow: 0 0 0 3px rgba(31, 143, 232, 0.2);
}

.image-item-large.selected::after {
  content: '✓';
  position: absolute;
  font-size: 40px;
  color: #1f8fe8;
  font-weight: bold;
}

.image-display {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.image-icon-large {
  font-size: 64px;
}

.image-file {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.challenge-actions {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
}

.btn-skip,
.btn-verify {
  padding: 10px 24px;
  border: 1px solid #d3d3d3;
  border-radius: 2px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  background: white;
  color: #1f8fe8;
}

.btn-skip:hover {
  border-color: #999;
  background: #f9f9f9;
}

.btn-verify {
  background: #1f8fe8;
  border-color: #1f8fe8;
  color: white;
}

.btn-verify:hover {
  background: #1676c9;
  border-color: #1676c9;
}

.error-message {
  margin-top: 15px;
  padding: 12px;
  background: #ffebee;
  border: 1px solid #ffcdd2;
  border-radius: 3px;
  color: #c62828;
  font-size: 13px;
  text-align: center;
  animation: shake 0.5s ease;
}

/* Success Section */
.success-section {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  padding: 40px;
  min-height: 140px;
  flex-direction: column;
}

.success-icon {
  font-size: 48px;
  color: #1f8fe8;
  font-weight: bold;
}

.success-text {
  text-align: center;
}

.success-title {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: #333;
}

.success-sub {
  margin: 4px 0 0 0;
  font-size: 13px;
  color: #5e5e5e;
}

/* Footer */
.captcha-footer {
  text-align: center;
  padding: 15px;
  font-size: 12px;
  color: #999;
}

.captcha-footer p {
  margin: 0;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

@keyframes shake {
  0%, 100% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-8px);
  }
  75% {
    transform: translateX(8px);
  }
}

@media (max-width: 600px) {
  .main-section {
    flex-direction: column;
    padding: 20px;
    gap: 15px;
  }

  .main-label {
    font-size: 18px;
  }

  .image-grid-large {
    grid-template-columns: repeat(2, 1fr);
  }

  .image-icon-large {
    font-size: 40px;
  }
}
</style>
