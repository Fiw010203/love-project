<script setup>
import { ref, onMounted } from 'vue'

const answered = ref(false)
const accepted = ref(false)

// --- ส่วนของการเล่นเพลงจาก YouTube ---
const videoId = "lvXILr0sISQ" 
const player = ref(null)
const volume = ref(50)

// --- ส่วนของ Google Forms ---
const FORM_ID = "1FAIpQLSer-kbKx8NNCY7jIlcsa5kq3cDvFOsNwEVHS3NWb7xrENGmhQ" 
const ENTRY_ID = "entry.1791752148"
const userComment = ref("")
const isSent = ref(false)
const isFlying = ref(false) // สถานะอนิเมชันซองจดหมาย


// ฟังก์ชันโหลด API และสร้าง Player แบบ Muted เพื่อให้ Autoplay บนมือถือผ่านง่ายขึ้น
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
      'playsinline': 1
    },
    events: {
      'onReady': (event) => {
        event.target.setVolume(volume.value)
        event.target.playVideo()
        event.target.unMute() // เปิดเสียงเมื่อ Player พร้อม
      }
    }
  })
}

async function sendToSheet() {
  if (!userComment.value.trim()) return
  
  // เริ่มอนิเมชันบิน
  isFlying.value = true
  
  const formData = new FormData()
  formData.append(ENTRY_ID, userComment.value)

  fetch(`https://docs.google.com/forms/d/e/${FORM_ID}/formResponse`, {
    method: 'POST',
    body: formData,
    mode: 'no-cors'
  })
  
  // รอให้อนิเมชันเล่นจบ (1 วินาที) แล้วค่อยเปลี่ยนสถานะเป็นส่งสำเร็จ
  setTimeout(() => {
    isSent.value = true
    isFlying.value = false
  }, 1000)
}

// ฟังก์ชันเมื่อกดตกลง
function sayYes() {
  accepted.value = true
  answered.value = true
  initYouTubePlayer() // เรียกเล่นเพลงทันทีที่ User Click
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
        <h1 class="main-title">
          เราอยากคุยกับเธอแบบจริงจังมากขึ้น<br />
          ให้มันชัดเจนขึ้นสำหรับเราสองคน<br />
          และถ้ามันเป็นไปได้…<br />
          <span class="highlight">เราก็อยากให้มันไปไกลกว่านั้น 💕</span><br />
          <span class="sub-question">เราลองเริ่มแบบนี้ด้วยกันได้มั้ย?</span>
        </h1>
        <div class="btn-group">
          <button class="yes-btn" @click="sayYes">ตกลง ✨</button>
          <button class="no-btn" :style="noStyle" @mouseover="moveNo" @touchstart.prevent="moveNo">ไม่</button>
        </div>
      </div>

      <div v-else :key="'result'" class="content success-page">
        <div class="floating-hearts">
          <span v-for="n in 12" :key="n">❤️</span>
        </div>
        
        <div class="success-text-group">
          <h1 class="bounce result-title">
            เย้! ขอบคุณที่ให้โอกาสเรานะ ❤️<br />
            <small>จากนี้…</small><br />
            เราจะตั้งใจดูแลความรู้สึกนี้ให้ดีที่สุดเลย
          </h1>
          
          <div class="form-container" v-if="!isSent">
  <textarea v-model="userComment" placeholder="พิมพ์ความในใจบอกเราหน่อย..." class="input-box"></textarea>
  
  <div class="button-wrapper">
    <button class="send-btn" @click="sendToSheet">ส่งความรู้สึก ✉️</button>
    
    <div v-if="isFlying" class="envelope-fly">📩</div>
  </div>
</div>
          
          <div class="volume-control">
            <span>🔈</span>
            <input type="range" min="0" max="100" v-model="volume" @input="updateVolume" />
            <span>🔊</span>
          </div>
          <p class="song-title">🎵 จีบ - QLER</p>
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;700&display=swap');

.button-wrapper {
  position: relative; /* เพื่อให้ซองจดหมายอ้างอิงตำแหน่งจากปุ่ม */
  display: flex;
  justify-content: center;
  width: 100%;
}

.envelope-fly {
  position: absolute;
  font-size: 30px;
  top: 0;
  z-index: 100;
  pointer-events: none;
  animation: flyAway 1s forwards ease-in;
}

@keyframes flyAway {
  0% {
    transform: translate(0, 0) scale(1) rotate(0deg);
    opacity: 1;
  }
  20% {
    transform: translate(-10px, 10px) scale(1.1) rotate(-10deg);
  }
  100% {
    transform: translate(200px, -400px) scale(0.5) rotate(45deg);
    opacity: 0;
  }
}

/* ปรับปรุง CSS หน้า Success เดิมเล็กน้อย */
.sent-msg {
  color: white;
  background: rgba(255, 255, 255, 0.3);
  padding: 15px 30px;
  border-radius: 25px;
  display: inline-block;
  margin-top: 20px;
  backdrop-filter: blur(5px);
  animation: popIn 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

@keyframes popIn {
  from { transform: scale(0.8); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

.love-container {
  position: fixed;
  inset: 0;
  background: linear-gradient(180deg, #FFD1D1 0%, #FFB6C1 100%);
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

.success-text-group {
  position: relative;
  z-index: 50;
}

.floating-hearts {
  position: absolute;
  inset: 0;
  z-index: 5;
  pointer-events: none;
}

/* Typography หน้าแรก */
.main-title {
  color: white;
  font-size: 1.4rem;
  line-height: 1.8;
  font-weight: 400;
  margin-bottom: 40px;
  text-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.main-title .highlight {
  font-weight: 500;
  display: block;
  margin: 10px 0;
}

.main-title .sub-question {
  font-size: 1.6rem;
  font-weight: 700;
  display: block;
  margin-top: 15px;
}

/* Typography หน้า Success */
.result-title {
  color: white;
  font-size: 1.6rem;
  line-height: 1.6;
  margin-bottom: 15px;
  font-weight: 500;
}

.result-title small {
  font-size: 1.1rem;
  opacity: 0.9;
  display: block;
  margin: 10px 0;
}

.sub-text {
  color: white;
  font-size: 1.2rem;
  font-weight: 300;
  letter-spacing: 1px;
  margin-top: 20px;
}

/* Form Styling */
.form-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
  margin: 20px 0;
}

.input-box {
  width: 85%;
  max-width: 300px;
  height: 80px;
  padding: 12px;
  border-radius: 15px;
  border: 2px solid rgba(255,255,255,0.5);
  background: rgba(255,255,255,0.2);
  color: white;
  outline: none;
  resize: none;
  font-family: 'Kanit';
}

.input-box::placeholder { color: rgba(255,255,255,0.7); }

.send-btn {
  background: #FF4D6D;
  color: white;
  border: none;
  padding: 10px 25px;
  border-radius: 50px;
  cursor: pointer;
  font-weight: bold;
}

/* ปุ่มและตัวควบคุม */
.btn-group {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  min-height: 80px;
}

button {
  font-size: 1.1rem;
  padding: 12px 35px;
  border-radius: 50px;
  border: none;
  font-weight: 500;
  cursor: pointer;
  box-shadow: 0 10px 20px rgba(0,0,0,0.05);
}

.yes-btn { background: #FF4D6D; color: white; }
.no-btn { background: #ffffff; color: #888; }

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

input[type="range"] { accent-color: #FF4D6D; cursor: pointer; }
.song-title { font-size: 0.85rem; color: white; opacity: 0.9; }

/* Animations */
@keyframes heartbeat {
  0%, 30%, 60% { transform: scale(1); }
  15% { transform: scale(1.15); }
}
.icon-header { font-size: 70px; animation: heartbeat 1.5s infinite; }

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-15px); }
}
.bounce { animation: bounce 1s infinite; }

@keyframes floatUp {
  0% { transform: translateY(0) rotate(0deg); opacity: 0; }
  20% { opacity: 1; }
  100% { transform: translateY(-110vh) rotate(360deg); opacity: 0; }
}

.floating-hearts span {
  position: absolute;
  bottom: -50px;
  font-size: 24px;
  animation: floatUp 4s linear infinite;
  opacity: 0;
}

.floating-hearts span:nth-child(1) { left: 10%; animation-delay: 0s; }
.floating-hearts span:nth-child(2) { left: 25%; animation-delay: 1s; }
.floating-hearts span:nth-child(3) { left: 40%; animation-delay: 2s; }
.floating-hearts span:nth-child(4) { left: 55%; animation-delay: 0.5s; }
.floating-hearts span:nth-child(5) { left: 70%; animation-delay: 1.5s; }
.floating-hearts span:nth-child(6) { left: 85%; animation-delay: 2.5s; }

.pop-enter-active, .pop-leave-active { transition: all 0.5s ease; }
.pop-enter-from { opacity: 0; transform: scale(0.8); }
.pop-leave-to { opacity: 0; transform: scale(1.2); }
</style>