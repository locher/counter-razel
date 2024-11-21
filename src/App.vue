<script setup>
import { ref } from 'vue'
import ConfettiExplosion from 'vue-confetti-explosion'

const urlParams = new URLSearchParams(window.location.search)
const numbersOfParticipants = urlParams.get('p') || 500

const numbers = ref(Array.from({ length: numbersOfParticipants }, (_, i) => i + 1))
const drawnNumber = ref(null)
const displayNumber = ref(null)
const confetti = ref(false)

const drawNumber = () => {
  confetti.value = false
  const randomIndex = Math.floor(Math.random() * numbers.value.length)
  drawnNumber.value = numbers.value.splice(randomIndex, 1)[0]

  let count = 0
  let speed = 50

  const displayNextNumber = () => {
    displayNumber.value = Math.floor(Math.random() * numbers.value.length) + 1
    count++

    if (count >= 25) {
      displayNumber.value = drawnNumber.value
      confetti.value = true
    } else {
      speed *= 1.09
      setTimeout(displayNextNumber, speed)
    }
  }

  displayNextNumber()
}
</script>

<template>
  <div class="screen">
    <img src="@/assets/background.svg" alt="" />
    <button @click="drawNumber">Tirer au sort</button>
    <p v-if="displayNumber !== null" class="number">{{ displayNumber }}</p>
    <img v-else src="@/assets/virginie.svg" alt="" />
    <ConfettiExplosion
      v-if="confetti"
      class="confetti"
      :colors="['#0056A3', '#D9CB1A', '#fff', '#000']"
      :particleCount="500"
      :stageHeight="1500"
      :stageWidth="2400"
    />
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Leckerli+One&display=swap');

body,
html {
  overflow: hidden;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.screen {
  width: 100vw;
  height: 100vh;
  background-color: white;
  position: relative;

  img {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translateX(-50%) translateY(-50%);
    width: 75vh;
    height: 75vh;
  }

  .number {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translateX(-50%) translateY(-50%);
    z-index: 3;
    font-size: 25vh;
    color: white;
    text-shadow: 0.2em 0.2em 0.4em rgba(0, 0, 0, 0.6);
    font-family: 'Leckerli One', cursive;
  }

  button {
    position: absolute;
    left: 50%;
    top: 70%;
    transform: translateX(-50%);
    z-index: 2;
    font-size: 1.5em;
    padding: 0.5em 1.5em;
    background-color: white;
    border-radius: 2em;
    cursor: pointer;
    font-family: 'Leckerli One', cursive;
    border: 0;
    color: #0056a3;
  }

  .confetti {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translateX(-50%) translateY(-50%);
    z-index: 2;
  }
}
</style>
