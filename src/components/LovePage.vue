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
      'playsinline': 1 // 👈 สำคัญมากสำหรับมือถือ เพื่อไม่ให้เด้งเข้าแอป/เต็มจอ
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
    <div id="youtube-player" style="position: absolute; bottom: 0; left: 0; width: 1px; height: 1px; opacity: 0; z-index: -1;"></div>

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
        
        <div class="success-text-group">
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
          
          <p class="song-title">
            🎵 อวดคนทั้งโลก (flex the world)
          </p>
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
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

.content { 
  text-align: center; 
  padding: 20px; 
  z-index: 10; /* ให้เนื้อหาอยู่ด้านบน */
  width: 100%;
}

/* จัดกลุ่มข้อความหน้า Success ให้อยู่เหนือหัวใจ */
.success-text-group {
  position: relative;
  z-index: 20;
}

.icon-header { font-size: 80px; margin-bottom: 10px; animation: heartbeat 1.5s infinite; }
h1 { color: white; font-size: 2.2rem; text-shadow: 0 2px 10px rgba(0,0,0,0.1); margin-bottom: 40px; }
.btn-group { display: flex; justify-content: center; align-items: center; gap: 20px; min-height: 100px; }

button {
  font-size: 1.3rem; padding: 12px 30px; border-radius: 50px; border: none;
  cursor: pointer; font-weight: bold; box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  transition: transform 0.2s;
}
.yes-btn { background: #FF4D6D; color: white; }
.no-btn { background: #ffffff; color: #888; }

.volume-control {
  margin: 20px auto;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  background: rgba(255, 255, 255, 0.4);
  padding: 8px 15px;
  border-radius: 20px;
  width: fit-content;
}

input[type="range"] { accent-color: #FF4D6D; }

.song-title { font-size: 0.8rem; opacity: 0.8; color: white; }

@keyframes heartbeat {
  0%, 30%, 60% { transform: scale(1); }
  15% { transform: scale(1.2); }
  45% { transform: scale(1.15); }
}
.bounce { animation: bounce 1s infinite; }
@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-15px); }
}

.pop-enter-active, .pop-leave-active { transition: all 0.5s ease; }
.pop-enter-from { opacity: 0; transform: scale(0.8); }
.pop-leave-to { opacity: 0; transform: scale(1.2); }

/* แก้ไขหัวใจให้ไปอยู่เลเยอร์หลังตัวหนังสือ */
.floating-hearts {
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 100%;
  z-index: 5; /* อยู่หลัง success-text-group */
  pointer-events: none; /* กดทะลุได้ */
}

.floating-hearts span {
  position: absolute; bottom: -50px;
  animation: float 3s linear infinite; opacity: 0;
}
@keyframes float {
  0% { transform: translateY(0) rotate(0deg); opacity: 1; }
  100% { transform: translateY(-110vh) rotate(360deg); opacity: 0; }
}

/* กระจายหัวใจให้ทั่วจอขึ้น */
.floating-hearts span:nth-child(1) { left: 5%; animation-delay: 0s; }
.floating-hearts span:nth-child(2) { left: 15%; animation-delay: 0.5s; }
.floating-hearts span:nth-child(3) { left: 25%; animation-delay: 1.2s; }
.floating-hearts span:nth-child(4) { left: 45%; animation-delay: 0.8s; }
.floating-hearts span:nth-child(5) { left: 65%; animation-delay: 1.5s; }
.floating-hearts span:nth-child(6) { left: 85%; animation-delay: 0.3s; }
.floating-hearts span:nth-child(7) { left: 95%; animation-delay: 1.1s; }
</style>