<script setup>
import { ref } from 'vue'
import ConfettiExplosion from 'vue-confetti-explosion'

// Shuffle array function
const shuffleArray = (array) => {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }

  return array
}

// Generate winners set
const generateWinners = () => {
  let winnersSet = new Set(forcedWinners.value ? forcedWinners.value.split(',').map(Number) : [])
  let loserSet = new Set(forcedLosers.value ? forcedLosers.value.split(',').map(Number) : [])

  while (winnersSet.size < numbersOfRolls.value) {
    const randomNum = Math.floor(Math.random() * numbersOfParticipants.value) + 1
    if (randomNum >= startWinners.value && !loserSet.has(randomNum)) {
      winnersSet.add(randomNum)
    }
  }

  return shuffleArray(Array.from(winnersSet))
}

// State variables
const numbersOfParticipants = ref(500)
const numbersOfRolls = ref(25)
const forcedWinners = ref('')
const forcedLosers = ref('')
const startWinners = ref('')
const drawnNumber = ref(null)
const displayNumber = ref(null)
const confetti = ref(false)
const isDrawing = ref(false)
const winners = ref(generateWinners())
const haveWin = ref([])
const isSettingsOpen = ref(false)

const winnersRestants = ref(winners.value)

const setWinners = () => {
  winners.value = generateWinners()
  winnersRestants.value = Array.from(winners.value)
  haveWin.value = []
}

// Draw number function
const drawNumber = () => {
  isDrawing.value = true
  haveWin.value.push(winnersRestants.value[0])
  drawnNumber.value = winnersRestants.value.shift()

  let count = 0
  let speed = 50

  const displayNextNumber = () => {
    displayNumber.value = Math.floor(Math.random() * numbersOfParticipants.value) + 1
    count++

    if (count >= 25) {
      displayNumber.value = drawnNumber.value
      confetti.value = true
      isDrawing.value = false
    } else {
      speed *= 1.09
      setTimeout(displayNextNumber, speed)
    }
  }

  displayNextNumber()
  confetti.value = false
}
</script>

<template>
  <div class="screen">
    <img src="@/assets/background.svg" alt="Background" />
    <Transition>
      <button
        v-show="winnersRestants.length > 0 && !isDrawing"
        @click="drawNumber"
        :disabled="isDrawing"
        class="drawn"
      >
        Tirer au sort
      </button>
    </Transition>
    <p v-if="displayNumber !== null" class="number">
      {{ displayNumber }}
    </p>
    <img v-else src="@/assets/virginie.svg" alt="Virginie" />
    <ConfettiExplosion
      v-if="confetti"
      class="confetti"
      :colors="['#0056A3', '#D9CB1A', '#fff', '#000']"
      :particleCount="400"
      :stageHeight="1500"
      :stageWidth="2400"
    />

    <button @click.prevent="isSettingsOpen = !isSettingsOpen" class="settingButton">
      Setttings
    </button>
  </div>

  <Transition>
    <div class="controls" v-if="isSettingsOpen">
      <label for="">
        Nombre de participants
        <input type="number" v-model="numbersOfParticipants" />
      </label>

      <label for="">
        Nombre de lots
        <input type="number" v-model="numbersOfRolls" />
      </label>

      <label for="">
        Gagnants forcés
        <input type="text" v-model="forcedWinners" />
      </label>

      <label for="">
        Perdants forcés
        <input type="text" v-model="forcedLosers" />
      </label>

      <label for="">
        Début gagnants
        <input type="number" v-model="startWinners" />
      </label>

      <button @click="setWinners">Calculer les gagnants</button>

      <p>Les gagants sont : {{ winners }}</p>
      <p>Le gagnants restants sont : {{ winnersRestants }}</p>
      <p>Ont déjà gagné : {{ haveWin }}</p>
    </div>
  </Transition>
</template>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Leckerli+One&display=swap');

body,
html {
  //overflow: hidden;
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
    transform: translate(-50%, -50%);
    width: 75vh;
    height: 75vh;
  }

  .number {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    z-index: 3;
    font-size: 25vh;
    color: white;
    text-shadow: 0.2em 0.2em 0.4em rgba(0, 0, 0, 0.6);
    font-family: 'Leckerli One', cursive;
  }

  .drawn {
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
    transform: translate(-50%, -50%);
    z-index: 2;
  }
}

.settingButton {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
}

.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}
</style>
