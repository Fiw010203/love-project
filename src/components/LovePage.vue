<script setup>
import { ref, watch } from 'vue'

const answered = ref(false)
const accepted = ref(false)

// --- ส่วนของการเล่นเพลงจาก YouTube ---
const videoId = "fNAMaJCwfLs" 
const player = ref(null)
const volume = ref(50)

// ฟังก์ชันโหลด API และสร้าง Player (ย้ายมาไว้ข้างนอกเพื่อให้เรียกใช้ได้ทันที)
function initYouTubePlayer() {
  if (!window.YT) {
    const tag = document.createElement('script')
    tag.src = "https://www.youtube.com/iframe_api"
    const firstScriptTag = document.getElementsByTagName('script')[0]
    firstScriptTag.parentNode.insertBefore(tag, firstScriptTag)
  }

  window.onYouTubeIframeAPIReady = () => {
    loadPlayer()
  }
  
  // ถ้า API โหลดมาอยู่แล้ว (เช่นเปลี่ยนหน้าไปมา)
  if (window.YT && window.YT.Player) {
    loadPlayer()
  }
}

function loadPlayer() {
  player.value = new window.YT.Player('youtube-player', {
    height: '0',
    width: '0',
    videoId: videoId,
    playerVars: {
      'autoplay': 1,
      'loop': 1,
      'playlist': videoId,
      'playsinline': 2 // 👈 สำคัญมากสำหรับมือถือ เพื่อไม่ให้เด้งเข้าแอป/เต็มจอ
    },
    events: {
      'onReady': (event) => {
        event.target.setVolume(volume.value)
        event.target.playVideo()
      }
    }
  })
}

// ฟังก์ชันเมื่อกดตกลง
function sayYes() {
  accepted.value = true
  answered.value = true
  // 👈 เรียกใช้งานเพลงทันทีที่ User Click เพื่อหลีกเลี่ยงการโดนบล็อก Autoplay บนมือถือ
  initYouTubePlayer()
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

function updateVolume() {
  if (player.value && player.value.setVolume) {
    player.value.setVolume(volume.value)
  }
}
</script>

<template>
  <div class="love-container">
    <div id="youtube-player" style="position: absolute; width: 0; height: 0; opacity: 0; pointer-events: none;"></div>

    <transition name="pop" mode="out-in">
      <div v-if="!answered" :key="'question'" class="content">
        <div class="icon-header">💖</div>
        <h1 class="main-title">เป็นแฟนกันนะ 💕</h1>
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
          <span v-for="n in 12" :key="n">❤️</span>
        </div>
        
        <div class="success-text-group">
          <h1 class="bounce result-title">เย้! รักกันนานๆ นะ ❤️</h1>
          <p class="sub-text">บันทึกไว้ในความทรงจำเรียบร้อย 😊</p>
          
          <div class="volume-control">
            <span>🔈</span>
            <input type="range" min="0" max="100" v-model="volume" @input="updateVolume" />
            <span>🔊</span>
          </div>
          <p class="song-title">🎵 อวดคนทั้งโลก (flex the world)</p>
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;500;700&display=swap');

.love-container {
  position: fixed;
  inset: 0;
  background-color: #FFD1D1;
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: 'Kanit', sans-serif;
  overflow: hidden;
}

.content {
  position: relative;
  text-align: center;
  width: 100%;
  padding: 20px;
  z-index: 10;
}

/* จัดการลำดับเลเยอร์หน้า Success */
.success-text-group {
  position: relative;
  z-index: 50; /* ให้ตัวหนังสืออยู่สูงกว่าหัวใจเสมอ */
}

/* หัวใจลอย (พื้นหลัง) */
.floating-hearts {
  position: absolute;
  inset: 0;
  z-index: 5; /* อยู่ต่ำกว่าตัวหนังสือ */
  pointer-events: none; /* ป้องกันหัวใจบังนิ้วตอนจะเลื่อนปรับเสียง */
}

/* ปรับขนาดตัวหนังสือให้เหมาะกับมือถือ */
.main-title {
  color: white;
  font-size: 2rem;
  margin-bottom: 40px;
  text-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.result-title {
  color: white;
  font-size: 1.8rem; /* เล็กลงนิดหน่อยเพื่อไม่ให้ล้นในมือถือ */
  line-height: 1.4;
  margin-bottom: 10px;
}

.sub-text {
  color: white;
  font-size: 1.1rem;
  margin-bottom: 20px;
}

/* ส่วนควบคุมเสียง */
.volume-control {
  margin: 20px auto;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  background: rgba(255, 255, 255, 0.4);
  padding: 10px 20px;
  border-radius: 30px;
  width: fit-content;
}

input[type="range"] {
  accent-color: #FF4D6D;
  cursor: pointer;
}

.song-title {
  font-size: 0.85rem;
  color: white;
  opacity: 0.9;
}

/* ปุ่มต่างๆ */
.btn-group {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  min-height: 80px;
}

button {
  font-size: 1.2rem;
  padding: 12px 30px;
  border-radius: 50px;
  border: none;
  font-weight: bold;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  transition: transform 0.2s;
}

.yes-btn { background: #FF4D6D; color: white; cursor: pointer; }
.no-btn { background: #ffffff; color: #888; }

/* Animations */
@keyframes heartbeat {
  0%, 30%, 60% { transform: scale(1); }
  15% { transform: scale(1.15); }
}
.icon-header { font-size: 70px; animation: heartbeat 1.5s infinite; }

.bounce { animation: bounce 1s infinite; }
@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-15px); }
}

.floating-hearts span {
  position: absolute;
  bottom: -50px;
  font-size: 24px;
  animation: floatUp 4s linear infinite;
  opacity: 0;
}

@keyframes floatUp {
  0% { transform: translateY(0) rotate(0deg); opacity: 0; }
  20% { opacity: 1; }
  100% { transform: translateY(-110vh) rotate(360deg); opacity: 0; }
}

/* กระจายหัวใจ */
.floating-hearts span:nth-child(1) { left: 10%; animation-delay: 0s; }
.floating-hearts span:nth-child(2) { left: 25%; animation-delay: 1s; }
.floating-hearts span:nth-child(3) { left: 40%; animation-delay: 2s; }
.floating-hearts span:nth-child(4) { left: 55%; animation-delay: 0.5s; }
.floating-hearts span:nth-child(5) { left: 70%; animation-delay: 1.5s; }
.floating-hearts span:nth-child(6) { left: 85%; animation-delay: 2.5s; }
</style>