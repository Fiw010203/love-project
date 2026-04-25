<script setup>
import { ref } from 'vue'

const emit = defineEmits(['next'])

const memories = [
  { text: "จำวันที่เราเริ่มคุยกันได้มั้ย 😊", img: "/images/1.jpg" },
  { text: "ตอนนั้นโคตรมีความสุขเลยนะ", img: "/images/2.jpg" },
  { text: "อยู่ด้วยแล้วมันสบายใจมาก ๆ 💖", img: "/images/3.jpg" },
  { text: "เลยอยากถามว่า...", img: "/images/4.jpg" }
]

const index = ref(0)

function next() {
  if (index.value < memories.length - 1) {
    index.value++
  } else {
    emit('next')
  }
}

function prev() {
  if (index.value > 0) index.value--
}

/* 👉 swipe logic */
let startX = 0
function touchStart(e) { startX = e.touches[0].clientX }
function touchEnd(e) {
  const endX = e.changedTouches[0].clientX
  const diff = endX - startX
  if (diff > 50) prev()
  if (diff < -50) next()
}

function clickZone(e) {
  const width = window.innerWidth
  if (e.clientX < width / 3) {
    prev()
  } else {
    next()
  }
}
</script>

<template>
  <div 
    class="story-container"
    @touchstart="touchStart"
    @touchend="touchEnd"
    @click="clickZone"
  >
    <div class="progress-container">
      <div 
        v-for="(m, i) in memories" 
        :key="i"
        class="progress-bar-bg"
      >
        <div 
          class="progress-fill"
          :class="{ 'active': i === index, 'completed': i < index }"
        ></div>
      </div>
    </div>

    <transition name="fade" mode="out-in">
      <img :key="index" :src="memories[index].img" class="main-image" />
    </transition>

    <div class="bottom-overlay"></div>

    <div class="text-wrapper">
      <transition name="slide-up" mode="out-in">
        <p :key="index" class="story-text">
          {{ memories[index].text }}
        </p>
      </transition>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500&display=swap');

.story-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: #000;
  color: white;
  font-family: 'Kanit', sans-serif;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

/* Progress Bar Style */
.progress-container {
  position: absolute;
  top: 15px;
  left: 10px;
  right: 10px;
  display: flex;
  gap: 6px;
  z-index: 20;
}

.progress-bar-bg {
  flex: 1;
  height: 3px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 10px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  width: 0%;
  background: white;
  transition: width 0.3s ease;
}

.progress-fill.completed {
  width: 100%;
}

.progress-fill.active {
  width: 100%; /* ในที่นี้ใช้การเปลี่ยน index ทันที ถ้าจะทำ auto-play ต้องใช้ animation */
}

/* Image Style */
.main-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  position: absolute;
  top: 0;
  left: 0;
}

/* Gradient Overlay */
.bottom-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 40%;
  background: linear-gradient(to top, rgba(0,0,0,0.7) 0%, transparent 100%);
  z-index: 5;
}

/* Text Style */
.text-wrapper {
  position: absolute;
  bottom: 80px;
  left: 0;
  right: 0;
  padding: 0 30px;
  z-index: 10;
  display: flex;
  justify-content: center;
}

.story-text {
  font-size: 24px;
  font-weight: 400;
  text-align: center;
  text-shadow: 0 2px 10px rgba(0,0,0,0.5);
  line-height: 1.5;
  margin: 0;
  /* เอฟเฟกต์กระจกเบลอเบาๆ */
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(8px);
  padding: 15px 25px;
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  width: fit-content;
}

/* Animations */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

.slide-up-enter-active {
  transition: all 0.4s ease-out;
}
.slide-up-enter-from {
  opacity: 0;
  transform: translateY(20px);
}
</style>