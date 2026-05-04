<template>
  <!-- Verification Before: Main Section -->
  <div v-if="!loading && !showChallenge && !verified" class="captcha-widget first-section" id="before-verification">
    <div class="checkbox-large">
      <div class="checkbox-wrapper">
        <input type="checkbox" id="captcha-checkbox" v-model="isChecked" @change="handleCheck">
      </div>
    </div>
    <div class="main-text-area">
      <label for="captcha-checkbox" class="main-label">
        私は3-Hです
      </label>
      <p class="sub-text">CSAM-CAPTCHA</p>
    </div>
  </div>

  <!-- Verification In Progress: Loading Section -->
  <div v-else-if="loading && !showChallenge && !verified" class="captcha-widget loading-section">
    <div class="spinner"></div>
    <p>検証中...</p>
  </div>

  <!-- Verification In Progress: Challenge Section -->
  <div v-else-if="showChallenge && !verified" class="captcha-widget challenge-section" id="challenge-section">
    <div class="challenge-header">
      <h3>{{ currentChallenge.prompt }}</h3>
      <button class="close-btn" @click="resetCaptcha">✕</button>
    </div>

    <div class="progress-bar-container">
      <div class="progress-bar">
        <div class="progress-fill" :style="{ width: ((currentProblemIndex + 1) / problems.length * 100) + '%' }"></div>
      </div>
      <p class="progress-text">{{ currentProblemIndex + 1 }} / {{ problems.length }}</p>
    </div>

    <div class="challenge-instruction">
      <p>{{ currentChallenge.instruction }}</p>
    </div>

    <div class="image-grid-large">
      <div v-for="(image, index) in currentChallenge.images" :key="index" class="image-item-large"
        :class="{ selected: selectedImages.includes(index) }" @click="toggleImage(index)">
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

  <!-- Verification Complete: Success Section -->
  <div v-else-if="verified" class="captcha-widget success-section" id="success-section">
    <div class="success-icon">✓</div>
    <div class="success-text">
      <p class="success-title">検証成功！</p>
      <p class="success-sub">あなたは今日から3-Hの仲間入りです！</p>
    </div>
    <button class="btn-verify" @click="resetCaptcha">もう一度プレイする</button>
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
const currentProblemIndex = ref(0)

// 問題1の画像ファイル情報を定義
const correctImages1 = [
  'a.jpg', 'b.jpg', 'c.jpg', 'd.jpg', 'e.jpg', 'f.jpg'
]

const wrongImages1 = [
  'a.jpg', 'b.jpg', 'c.jpg', 'd.jpg', 'e.jpg', 'f.jpg'
]

// 問題2の画像ファイル情報を定義
const correctImages2 = [
  'a.png', 'b.png', 'c.png', 'd.png', 'e.png', 'f.png', 'g.png', 'h.png'
]

const wrongImages2 = [
  'a.png', 'b.png', 'c.png', 'd.png', 'e.png', 'f.png', 'g.png', 'h.png',
  'i.png', 'j.png', 'k.png', 'l.png', 'm.png', 'n.png', 'o.png', 'p.png',
  'q.png', 'r.png', 's.png', 't.png'
]

// 問題3の画像ファイル情報を定義
const correctImages3 = [
  'a.png', 'b.png', 'c.png', 'd.png', 'e.png', 'f.png', 'g.png', 'h.png'
]

const wrongImages3 = [
  'a.png', 'b.png', 'c.png', 'd.png', 'e.png', 'f.png', 'g.png', 'h.png',
  'i.png', 'j.png', 'k.png', 'l.png', 'm.png', 'n.png', 'o.png', 'p.png',
  'q.png', 'r.png', 's.png', 't.png'
]

const problems = [
  {
    prompt: '3-Hに当てはまるのは？',
    instruction: '3-H全体に当てはまるものをもれなく選択してください',
    basePath: '/images/problem1',
    correctImages: correctImages1,
    wrongImages: wrongImages1
  },
  {
    prompt: '1はどれか？',
    instruction: '数字の1を含む画像をすべてクリックしてください②',
    basePath: '/images/problem2',
    correctImages: correctImages2,
    wrongImages: wrongImages2
  },
  {
    prompt: '1はどれか？',
    instruction: '数字の1を含む画像をすべてクリックしてください③',
    basePath: '/images/problem3',
    correctImages: correctImages3,
    wrongImages: wrongImages3
  }
]

const generateChallenge = (problemIdx = currentProblemIndex.value) => {
  // 指定された問題を使用
  const currentProblem = problems[problemIdx]

  // 正解の数をランダムに生成（3～6）
  const correctCount = Math.floor(Math.random() * 4) + 3

  // 正解画像をランダムに選択
  const selectedCorrectImages = []
  const tempCorrectImages = [...currentProblem.correctImages]
  for (let i = 0; i < correctCount; i++) {
    const randomIdx = Math.floor(Math.random() * tempCorrectImages.length)
    selectedCorrectImages.push(tempCorrectImages[randomIdx])
    tempCorrectImages.splice(randomIdx, 1)
  }

  // 不正解画像を選択（9-correctCount個）
  const selectedWrongImages = []
  const tempWrongImages = [...currentProblem.wrongImages]
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
    prompt: currentProblem.prompt,
    instruction: currentProblem.instruction,
    images: images.map(img => ({
      src: `${currentProblem.basePath}/${img.isCorrect ? 'correctImgs' : 'wrongImgs'}/${img.filename}`,
      alt: "inspection_image"
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

  // 正解した場合
  if (currentProblemIndex.value < problems.length - 1) {
    // 次の問題がある場合
    currentProblemIndex.value++
    currentChallenge.value = generateChallenge()
    selectedImages.value = []
    error.value = ''
  } else {
    // すべての問題が完了した場合
    verified.value = true
    emit('verified', true)
    setTimeout(() => {
      showChallenge.value = false
    }, 1500)
  }
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
  currentProblemIndex.value = 0
  currentChallenge.value = generateChallenge()
  emit('verified', false)
}
</script>

<style scoped>
.captcha-widget {
  margin: 0;
  padding: 2vh;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  margin-left: auto;
  margin-right: auto;
  background: white;
  border: 1px solid #9e9e9e;
  border-radius: 2px;
  box-shadow: 0 0.2vh 1vh rgba(0, 0, 0, 0.12);
}

/* First Section */
.first-section {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  gap: 4vw;
  padding: 4vh 5vw;
  min-height: auto;
  width: 85%;
  height: 150px;
  max-width: 600px;
}

.checkbox-large {
  flex-shrink: 0;
}

.checkbox-wrapper {
  width: 6vw;
  height: 6vw;
  min-width: 44px;
  min-height: 44px;
  max-width: 60px;
  max-height: 60px;
}

.checkbox-large input[type="checkbox"] {
  width: 100%;
  height: 100%;
  cursor: pointer;
  accent-color: #1f8fe8;
}

.checkbox-spinner {
  width: 6vw;
  height: 6vw;
  min-width: 44px;
  min-height: 44px;
  max-width: 60px;
  max-height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.spinner-checkbox {
  width: 100%;
  height: 100%;
  border: 3px solid #e8e8e8;
  border-top-color: #1f8fe8;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  box-sizing: border-box;
}

.main-text-area {
  width: fit-content;
}

.main-label {
  display: block;
  font-size: 1.5em;
  font-weight: 500;
  color: #333;
  margin-bottom: 0.5vh;
  cursor: pointer;
  user-select: none;
  line-height: 1.3;
  width: fit-content;
}

.sub-text {
  margin: 0;
  font-size: clamp(12px, 2vw, 15px);
  color: #999;
  display: block;
  width: fit-content;
}

.logo-area {
  flex-shrink: 0;
  width: 10vw;
  height: 10vw;
  min-width: 56px;
  max-width: 100px;
  display: none;
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
  gap: 2vh;
  padding: 4vh;
  min-height: 30vh;
  min-width: 250px;
}

.spinner {
  width: 10vw;
  height: 10vw;
  max-width: 60px;
  max-height: 60px;
  border: 3px solid #f0f0f0;
  border-top-color: #1f8fe8;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  box-sizing: border-box;
}

.loading-section p {
  margin: 0;
  font-size: clamp(13px, 2vw, 15px);
  color: #666;
}

/* Challenge Section */
.challenge-section {
  padding: 3vh;
  width: 85%;
  max-width: 500px;
  display: flex;
  flex-direction: column;
  gap: 2vh;
  max-height: 90vh;
  overflow-y: auto;
}

.challenge-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 1.5vw;
  flex-shrink: 0;
}

.challenge-header h3 {
  margin: 0;
  font-size: clamp(16px, 5vw, 18px);
  font-weight: 600;
  color: #333;
  line-height: 1.4;
}

.close-btn {
  background: none;
  border: none;
  font-size: clamp(18px, 4vw, 24px);
  color: #999;
  cursor: pointer;
  padding: 0.5vh 0.8vw;
  min-width: 40px;
  min-height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.2s;
  flex-shrink: 0;
  -webkit-touch-callout: none;
  -webkit-user-select: none;
}

.close-btn:hover,
.close-btn:active {
  color: #333;
}

.progress-bar-container {
  flex-shrink: 0;
  margin-bottom: 1.5vh;
}

.progress-bar {
  width: 100%;
  height: 6px;
  background: #e8e8e8;
  border-radius: 3px;
  overflow: hidden;
  margin-bottom: 0.8vh;
}

.progress-fill {
  height: 100%;
  background: #1f8fe8;
  transition: width 0.3s ease;
}

.progress-text {
  text-align: center;
  font-size: clamp(11px, 1.8vw, 12px);
  color: #666;
  margin: 0;
}

.challenge-instruction {
  text-align: center;
  flex-shrink: 0;
}

.challenge-instruction p {
  margin: 0;
  font-size: clamp(13px, 3vw, 14px);
  color: #666;
  line-height: 1.5;
}

.image-grid-large {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: clamp(8px, 2vw, 15px);
  flex: 1;
  min-height: 0;
  aspect-ratio: 1;
}

.image-item-large {
  aspect-ratio: 1;
  border: clamp(2px, 0.5vw, 4px) solid #d3d3d3;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  background: #fafafa;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  position: relative;
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  box-sizing: border-box;
}

.image-item-large:active {
  border-color: #1f8fe8;
  background: #e8f4fb;
}

.image-item-large.selected {
  border-color: #1f8fe8;
  background: #e8f4fb;
  box-shadow: 0 0 0 clamp(2px, 0.5vw, 3px) rgba(31, 143, 232, 0.2);
}

.image-item-large.selected::after {
  content: '✓';
  position: absolute;
  font-size: clamp(24px, 6vw, 40px);
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
  font-size: clamp(48px, 12vw, 64px);
}

.image-file {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.challenge-actions {
  display: flex;
  gap: 1.5vw;
  justify-content: flex-end;
  flex-wrap: wrap;
  flex-shrink: 0;
}

.btn-skip,
.btn-verify {
  padding: clamp(10px, 1.5vh, 14px) clamp(16px, 3vw, 24px);
  border: 1px solid #d3d3d3;
  border-radius: 7px;
  font-size: clamp(12px, 2vw, 14px);
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  background: white;
  color: #1f8fe8;
  min-height: 44px;
  min-width: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  box-sizing: border-box;
}

.btn-skip:active {
  border-color: #999;
  background: #f9f9f9;
}

.btn-verify {
  background: #1f8fe8;
  border-color: #1f8fe8;
  color: white;
}

.btn-verify:active {
  background: #1676c9;
  border-color: #1676c9;
}

.error-message {
  margin-top: 1.5vh;
  padding: 1.5vh;
  background: #ffebee;
  border: 1px solid #ffcdd2;
  border-radius: 3px;
  color: #c62828;
  font-size: clamp(11px, 1.8vw, 13px);
  text-align: center;
  animation: shake 0.5s ease;
  line-height: 1.4;
  flex-shrink: 0;
}

/* Success Section */
.success-section {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 2vh;
  padding: 4vh;
  min-height: 25vh;
  flex-direction: column;
}

.success-icon {
  font-size: clamp(32px, 8vw, 48px);
  color: #1f8fe8;
  font-weight: bold;
}

.success-text {
  text-align: center;
}

.success-title {
  margin: 0;
  font-size: clamp(15px, 3vw, 18px);
  font-weight: 600;
  color: #333;
  line-height: 1.4;
}

.success-sub {
  margin: 0.5vh 0 0 0;
  font-size: clamp(11px, 2vw, 13px);
  color: #5e5e5e;
  line-height: 1.5;
}

/* Footer */
.captcha-footer {
  text-align: center;
  padding: 1.5vh;
  font-size: clamp(10px, 1.5vw, 12px);
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

  0%,
  100% {
    transform: translateX(0);
  }

  25% {
    transform: translateX(-0.8vh);
  }

  75% {
    transform: translateX(0.8vh);
  }
}

/* Tablet and Large Screens */
@media (min-width: 768px) {
  .logo-area {
    display: block;
  }

  .btn-skip:hover {
    border-color: #999;
    background: #f9f9f9;
  }

  .btn-verify:hover {
    background: #1676c9;
    border-color: #1676c9;
  }

  .image-item-large:hover {
    border-color: #999;
    box-shadow: 0 0.2vh 0.8vh rgba(0, 0, 0, 0.1);
  }
}
</style>
