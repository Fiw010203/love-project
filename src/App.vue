<script setup>
import { ref } from 'vue'
import Login from './components/Login.vue'
import Memory from './components/Memory.vue'
import LovePage from './components/LovePage.vue'

const step = ref(1)
</script>

<template>
  <div class="container">

    <Login v-if="step === 1" @success="step = 2" />

    <Memory v-if="step === 2" @next="step = 3" />

    <LovePage v-if="step === 3" />

  </div>
</template>
<style>
.container {
  background: #ffe4ec;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>