<script setup>
import { ref, watch } from 'vue'

const answered = ref(false)
const accepted = ref(false)

// ฟังก์ชันเมื่อกดตกลง
function sayYes() {
  accepted.value = true
  answered.value = true
}

const noStyle = ref({
  position: 'relative',
  transition: 'all 0.2s ease'
})

function moveNo() {
  const x = Math.random() * (window.innerWidth - 120)
  const y = Math.random() * (window.innerHeight - 60)
  noStyle.value = {
    position: 'fixed',
    left: x + 'px',
    top: y + 'px',
    zIndex: 999,
    transition: 'all 0.2s ease'
  }
}

// --- ส่วนของการเล่นเพลงจาก YouTube ---
// ใช้ Video ID จากลิงก์ที่คุณให้มา: fNAMaJCwfLs 
const videoId = "fNAMaJCwfLs" 
const player = ref(null)

// ตรวจสอบเมื่อตอบตกลงแล้ว ให้เริ่มโหลดวิดีโอ
watch(accepted, (newVal) => {
  if (newVal) {
    // สร้าง IFrame สำหรับเล่นเพลงแบบซ่อนไว้
    const tag = document.createElement('script')
    tag.src = "https://www.youtube.com/iframe_api"
    const firstScriptTag = document.getElementsByTagName('script')[0]
    firstScriptTag.parentNode.insertBefore(tag, firstScriptTag)

    window.onYouTubeIframeAPIReady = () => {
      player.value = new window.YT.Player('youtube-player', {
        height: '0',
        width: '0',
        videoId: videoId,
        playerVars: {
          'autoplay': 1,
          'loop': 1,
          'playlist': videoId
        },
        events: {
          'onReady': (event) => {
            event.target.playVideo()
          }
        }
      })
    }
  }
})
const volume = ref(50) // เริ่มต้นที่ความดัง 50%

// ฟังก์ชันสำหรับปรับระดับเสียง
function updateVolume() {
  if (player.value && player.value.setVolume) {
    player.value.setVolume(volume.value)
  }
}
</script>

<template>
  <div class="love-container">
    <div id="youtube-player" style="position: absolute; visibility: hidden;"></div>

    <transition name="pop" mode="out-in">
      <div v-if="!answered" :key="'question'" class="content">
        <div class="icon-header">💖</div>
        <h1>เป็นแฟนกันนะ 💕</h1>
        <div class="btn-group">
          <button class="yes-btn" @click="sayYes">ตกลง ✨</button>
          <button 
            class="no-btn" 
            :style="noStyle"
            @mouseover="moveNo"
            @touchstart.prevent="moveNo"
          >
            ไม่
          </button>
        </div>
      </div>

      <div v-else :key="'result'" class="content success-page">
  <div class="floating-hearts">
    <span v-for="n in 10" :key="n">❤️</span>
  </div>
  <h1 class="bounce">เย้! รักกันนานๆ นะ ❤️</h1>
  <p>บันทึกไว้ในความทรงจำเรียบร้อย 😊</p>
  
  <div class="volume-control">
    <span>🔈</span>
    <input 
      type="range" 
      min="0" 
      max="100" 
      v-model="volume" 
      @input="updateVolume"
    />
    <span>🔊</span>
  </div>
  
  <p style="font-size: 0.8rem; margin-top: 10px; opacity: 0.7;">
    🎵 อวดคนทั้งโลก (flex the world)
  </p>
</div>
    </transition>
  </div>
</template>

<style scoped>
/* ใช้ CSS เดิมจากที่คุณชอบได้เลยครับ */
@import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;500;700&display=swap');

.love-container {
  position: fixed;
  top: 0; left: 0;
  width: 100vw; height: 100vh;
  background-color: #FFD1D1;
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: 'Kanit', sans-serif;
  overflow: hidden;
}

.content { text-align: center; padding: 20px; z-index: 10; }
.icon-header { font-size: 80px; margin-bottom: 10px; animation: heartbeat 1.5s infinite; }
h1 { color: white; font-size: 2.5rem; text-shadow: 0 2px 10px rgba(0,0,0,0.1); margin-bottom: 40px; }
.btn-group { display: flex; justify-content: center; align-items: center; gap: 20px; min-height: 100px; }

button {
  font-size: 1.5rem; padding: 15px 40px; border-radius: 50px; border: none;
  cursor: pointer; font-weight: bold; box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  transition: transform 0.2s;
}
.yes-btn { background: #FF4D6D; color: white; }
.no-btn { background: #ffffff; color: #888; }

@keyframes heartbeat {
  0%, 30%, 60% { transform: scale(1); }
  15% { transform: scale(1.2); }
  45% { transform: scale(1.15); }
}
.bounce { animation: bounce 1s infinite; }
@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-20px); }
}

.pop-enter-active, .pop-leave-active { transition: all 0.5s ease; }
.pop-enter-from { opacity: 0; transform: scale(0.8); }
.pop-leave-to { opacity: 0; transform: scale(1.2); }

.floating-hearts span {
  position: absolute; bottom: -50px;
  animation: float 3s linear infinite; opacity: 0;
}
@keyframes float {
  0% { transform: translateY(0); opacity: 1; }
  100% { transform: translateY(-100vh); opacity: 0; }
}
.floating-hearts span:nth-child(1) { left: 10%; animation-delay: 0s; }
.floating-hearts span:nth-child(2) { left: 30%; animation-delay: 0.5s; }
.floating-hearts span:nth-child(3) { left: 50%; animation-delay: 1.2s; }
.floating-hearts span:nth-child(4) { left: 70%; animation-delay: 0.8s; }
.floating-hearts span:nth-child(5) { left: 90%; animation-delay: 1.5s; }
</style>