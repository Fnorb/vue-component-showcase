<template>
  <div class="app-container">
  </div>
  <HelloWorld msg="Vite + Vue" />
  <AnimatedBackground trackingElement=".app-container" />
</template>

<script setup>
import HelloWorld from './components/HelloWorld.vue'
import AnimatedBackground from './components/AnimatedBackground.vue'

import { ref } from 'vue'

const activateComponent = ref("animatedBackground")
</script>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
