<script setup>
import { ref } from 'vue'

const emit = defineEmits(['success'])

const correctPassword = "123"
const inputPassword = ref("")
const isWrong = ref(false)

function checkPassword() {
  if (inputPassword.value === correctPassword) {
    emit('success')
  } else {
    isWrong.value = true
    setTimeout(() => isWrong.value = false, 500)
    inputPassword.value = ""
  }
}
</script>

<template>
  <div class="login-container">
    <div class="content-wrapper" :class="{ 'shake': isWrong }">
      <h1>ใส่รหัสก่อนนะ 💌</h1>

      <div class="input-group">
        <input 
          v-model="inputPassword" 
          type="password"
          placeholder="ใส่รหัส..."
          @keyup.enter="checkPassword"
        />
        <button @click="checkPassword">เข้า</button>
      </div>

      <Transition name="fade">
        <p v-if="isWrong" class="error-msg">รหัสไม่ถูกนะ 😝</p>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
/* ส่วนสำคัญคือตรงนี้ครับ ต้องให้ container สูงและกว้างเต็มจอ */
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  
  /* ใช้ vh/vw เพื่อให้เต็มหน้าจอจริง ๆ */
  width: 100vw;
  height: 100vh;
  
  /* สีชมพูพื้นหลัง */
  background-color: #FFD1D1; /* ปรับเฉดชมพูอ่อนตามใจชอบ */
  
  /* ป้องกันไม่ให้มีขอบขาวรอบ ๆ */
  position: fixed;
  top: 0;
  left: 0;
  
  font-family: 'Kanit', sans-serif;
}

.content-wrapper {
  text-align: center;
  width: 100%;
  max-width: 350px; /* จำกัดความกว้างของปุ่มและช่องกรอก */
  padding: 20px;
}

h1 {
  color: white;
  font-size: 2.5rem;
  margin-bottom: 30px;
  text-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

input {
  padding: 18px;
  border-radius: 20px;
  border: none;
  outline: none;
  background: #FFF9F9;
  font-size: 18px;
  text-align: center;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
}

button {
  padding: 16px;
  border-radius: 20px;
  border: none;
  background: #FF8E8E; /* สีชมพูเข้มขึ้นมาหน่อยสำหรับปุ่ม */
  color: white;
  font-size: 20px;
  font-weight: bold;
  cursor: pointer;
  transition: transform 0.2s, background 0.3s;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.1);
}

button:hover {
  background: #FF7575;
  transform: translateY(-2px);
}

.error-msg {
  margin-top: 20px;
  color: white;
  font-weight: bold;
}
</style>