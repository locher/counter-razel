<script setup>
import { ref } from 'vue'
import ConfettiExplosion from 'vue-confetti-explosion'
import Setting from '@/icons/Setting.vue'
import Close from '@/icons/Close.vue'

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
  let maxIterations = 1000
  let iterations = 0

  while (winnersSet.size < numbersOfRolls.value && iterations < maxIterations) {
    const randomNum = Math.floor(Math.random() * numbersOfParticipants.value) + 1
    if (randomNum >= startWinners.value && !loserSet.has(randomNum)) {
      winnersSet.add(randomNum)
    }
    iterations++
  }

  if (iterations >= maxIterations) {
    console.warn('Max iterations reached, some winners might be missing.')
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

// Close overlay
const toggleOverlay = () => {
  isSettingsOpen.value = !isSettingsOpen.value
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

    <button @click="toggleOverlay" class="settingButton">
      <Setting />
    </button>
  </div>

  <Transition>
    <div>
      <div class="overlay" v-if="isSettingsOpen" @click="toggleOverlay"></div>
      <div class="controls" v-if="isSettingsOpen">
        <Close class="close" @click="toggleOverlay" />
        <label for="">
          Nombre de participants
          <input type="number" v-model="numbersOfParticipants" />
        </label>

        <label for="">
          Nombre de lots
          <input type="number" v-model="numbersOfRolls" />
        </label>

        <label for="">
          Gagnants forcés<span>Séparés par des virgules</span>
          <input type="text" v-model="forcedWinners" />
        </label>

        <label for="">
          Perdants forcés<span>Séparés par des virgules</span>
          <input type="text" v-model="forcedLosers" />
        </label>

        <label for="">
          Début gagnants<span>Le nombre renseigné pourra gagner</span>
          <input type="number" v-model="startWinners" />
        </label>

        <button @click="setWinners">Calculer les gagnants</button>

        <div class="overlay__infos">
          <p>Les gagants sont : {{ winners }}</p>
          <p v-if="winnersRestants.length > 0">Le gagnants restants sont : {{ winnersRestants }}</p>
          <p v-if="haveWin.length > 0">Ont déjà gagné : {{ haveWin }}</p>
        </div>
      </div>
    </div>
  </Transition>
</template>

<style lang="scss">
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
    font-size: 1.2vw;
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

.controls {
  position: fixed;
  top: 50%;
  left: 50%;
  background-color: white;
  transform: translateX(-50%) translateY(-50%);
  z-index: 200;
  padding: 2rem;
  border-radius: 1rem;
  box-shadow: 0 0 1rem rgba(0, 0, 0, 0.2);
  display: grid;
  gap: 1rem;
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  font-size: 14px;
  width: 40vw;

  label {
    input {
      display: block;
      margin-top: 0.35em;
    }

    span {
      font-size: 12px;
      color: #959595;
      padding-left: 1em;
    }
  }

  button {
    padding: 0.5em 1em;
    background-color: #0056a3;
    color: white;
    border: 0;
    border-radius: 0.5em;
    cursor: pointer;
    justify-self: baseline;
  }
}

.overlay {
  background-color: rgba(#0056a3, 0.5);
  filter: blur(6px);
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  cursor: pointer;
  z-index: 100;

  &__infos {
    display: grid;
    gap: 0.8rem;

    p {
      font-size: 12px;
    }
  }
}

.close {
  position: fixed;
  top: 2rem;
  right: 2rem;
  height: 2rem;
  width: 2rem;
  fill: grey;
}

.settingButton {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
  background: none;
  border: 0;
  cursor: pointer;

  svg {
    height: 1.5rem;
    width: 1.5rem;
    fill: lightgrey;
  }
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
