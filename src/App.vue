<script setup>
import { ref, onMounted, computed } from "vue"
import bgSound from "./assets/audio/bg-sound.mp3"
import clickSound from "./assets/audio/click-sound.mp3"

const numbers = ref(Array.from(Array(51).keys()).splice(1))
const usedNumber = ref([])
const randomBtnText = ref("Start Bingo Game")
const toDisabledwhileRandom = ref(false)
const shuffledNumbers = ref([])
const selectedNumbers = ref([])
const showAudio = ref(true)
const level = ref("default")
const gameStart = ref(false)
const showHiddenNumbers = ref(false)
const countdown = ref(1)
const alertCountdown = ref(false)
const showCard1 = ref(false)
const showCard2 = ref(false)

let autoRandomInterval = null

const setLevel = (newLevel) => {
  gameStart.value = true
  level.value = newLevel
}

onMounted(() => {
  shuffleNumber()
})

const randomNumber = () => {
  let randomIndex = Math.floor(Math.random() * numbers.value.length)
  let number = numbers.value.splice(randomIndex, 1)[0]
  usedNumber.value.push(number)

  if (numbers.value.length === 0) {
    randomBtnText.value = "Out Of Number!"
    clearInterval(autoRandomInterval)
  }
  toDisabledwhileRandom.value = true
}

let numberOnBoard = Array.apply(null, { length: 51 }).map(Number.call, Number)
numberOnBoard.shift()

const shuffleNumber = () => {
  let currentIndex = numberOnBoard.length,
    randomIndex

  while (currentIndex != 0) {
    randomIndex = Math.floor(Math.random() * currentIndex)
    currentIndex--
    ;[numberOnBoard[currentIndex], numberOnBoard[randomIndex]] = [
      numberOnBoard[randomIndex],
      numberOnBoard[currentIndex]
    ]
  }
  shuffledNumbers.value = [...numberOnBoard]
  toDisabledwhileRandom.value = false
}

console.log(numberOnBoard)

const toggleSelection = (number) => {
  if (usedNumber.value.includes(number)) {
    // if (number === usedNumber.value[usedNumber.value.length - 1])
    if (selectedNumbers.value.includes(number)) {
      selectedNumbers.value = selectedNumbers.value.filter(
        (num) => num !== number
      )
    } else {
      selectedNumbers.value.push(number)
      clickMusic()
    }
  }
}

const isSelected = (number) => {
  return selectedNumbers.value.includes(number)
}

console.log("selectedNumbers.value", selectedNumbers.value)

const bgmusic = new Audio(bgSound)
const toggleSound = new Audio(clickSound)

const playMusic = () => {
  bgmusic.play()
  showAudio.value = !showAudio.value
}

const stopMusic = () => {
  bgmusic.pause()
  showAudio.value = !showAudio.value
}

const clickMusic = () => {
  toggleSound.play()
}

const visibleNumbers = computed(() => {
  return usedNumber.value.slice(-5)
})

const hiddenNumbers = computed(() => {
  return usedNumber.value.slice(0)
})

console.log(visibleNumbers)
console.log(shuffleNumber())

const checkLineWin = () => {
  // Check rows
  for (let row = 0; row < 5; row++) {
    let allMarked = true
    for (let col = 0; col < 5; col++) {
      if (!selectedNumbers.value[row * 5 + col]) {
        allMarked = false
        break
      }
    }
    if (allMarked) return true

  }


  // Check columns
  for (let col = 0; col < 5; col++) {
    let allMarked = true
    for (let row = 0; row < 5; row++) {
      if (!selectedNumbers.value[row * 5 + col]) {
        allMarked = false
        break
      }
    }
    if (allMarked) return true
  }

  // Check diagonals
  let allMarked = true
  for (let i = 0; i < 5; i++) {
    if (!selectedNumbers.value[i * 5 + i]) {
      allMarked = false
      break
    }
  }
  if (allMarked) return true

  // Check Anti-diagonal
  allMarked = true
  for (let i = 0; i < 5; i++) {
    if (!selectedNumbers.value[i * 5 + (4 - i)]) {
      allMarked = false
      break
    }
  }
  if (allMarked) return true

  return false
}

const checkBlackoutWin = () => {
  return selectedNumbers.value.length === 25 //ถ้าเลขที่เลือกเท่ากับ 25 ตัว
}


const hasWon = computed(() => {
  switch (level.value) {
    case "line":
      return checkLineWin()
    case "blackout":
      return checkBlackoutWin()
    default:
      return false
  }
})

// auto-random-number
const startAutoRandomNumber = () => {
  toDisabledwhileRandom.value = true
  randomBtnText.value = "Randomizing..."

  autoRandomInterval = setInterval(() => {
    randomNumber()
  }, 1)

  alertCountdown.value = true
  startCountdown()
}

const startCountdown = () => {
  let interval = setInterval(() => {
    countdown.value--
    if (countdown.value <= 0) {
      clearInterval(interval)
      alertCountdown.value = false
    }
  }, 1000)
}

const resetGame = () => {
  clearInterval(intervalId)
  numbers.value = Array.from({ length: 51 }, (_, i) => i + 1)
  usedNumber.value = []
  selectedNumbers.value = []
  shuffleNumber()
  randomBtnText.value = "Random Number"
  level.value = "default"
  showHiddenNumbers.value = false
  hasWon.value = false // ซ่อนโมดอลเมื่อเริ่มเกมใหม่
}
</script>

<template>
  <div class="relative w-full h-full">
    <!-- Video Background -->
    <video
      class="absolute top-0 left-0 w-full object-cover h-screen"
      autoplay
      loop
      muted
    >
      <source src="./assets/video/bgbingo.mp4" type="video/mp4" />
    </video>

    <!-- Level Selection -->
    <div class="absolute inset-0 flex flex-col mt-28 ml-4" v-if="!gameStart">
      <!-- Logo Level -->
      <!-- <div class="flex items-center">
        <img class="h-40 w-40" src="../src/assets/img/bingo.png" />

      </div> -->

      <!-- <div class="card bg-base-100 w-96 shadow-xl">
        <div class="card-body items-center text-center">
          <h1 class="text-center">SELECT BINGO MODE</h1>
          <div class="card-actions justify-end">
            <button class="btn btn-primary" @click="setLevel('line')">Line</button>
            <button class="btn btn-error"  @click="setLevel('blackout')"> Black Out</button>
          </div>
        </div>
      </div> -->

      <div class="text-focus-in">
        <div class="flex items-center justify-between">
          <div class="mt-12">
            <h1 class="text-7xl m-5">Welcome to your Bingo!</h1>
            <h1 class="text-7xl m-5">Challenge yourself</h1>
            <h1 class="text-4xl text-left m-5">Start playing now!</h1>
            <div class="flex flex-row">
              <div ontouchstart="">
                <div
                  class="button"
                  @mouseover="showCard1 = true"
                  @mouseout="showCard1 = false"
                >
                  <a @click="setLevel('line')">Line</a>
                </div>
              </div>
              <div ontouchstart="">
                <div
                  class="button"
                  @mouseover="showCard2 = true"
                  @mouseout="showCard2 = false"
                >
                  <a @click="setLevel('blackout')">Black Out</a>
                </div>
              </div>
            </div>
          </div>
          <!-- v-if="showCard1" -->
          <div class="flex justify-end" v-if="showCard1">
            <div class="card bg-base-100 w-96 shadow-xl mr-24">
              <div class="card-body">
                <h2 class="card-title text-2xl">Line Win Pattern</h2>
              </div>
              <figure>
                <img src="./assets/img/bingoWinLine1.png" alt="Shoes" />
              </figure>
              <figure>
                <img class="w-80" src="./assets/img/bingoWinLine2.png" alt="Shoes" />
              </figure>
            </div>
          </div>

          <div class="flex justify-end" v-if="showCard2">
            <div class="card bg-base-100 w-96 shadow-xl mr-24">
              <div class="card-body">
                <h2 class="card-title text-2xl">Blackout Win Pattern</h2>
              </div>
              <figure>
                <img  class=" w-96" src="./assets/img/bingoWinBlackout.png" alt="Shoes" />
              </figure>
             
            </div>
          </div>
        </div>

      </div>
    </div>

    <!-- game content -->

    <div v-if="gameStart" class="relative z-10 flex flex-row w-full h-full">
      <div class="w-1/2 flex flex-col justify-center items-center">
        <!-- Audio Controls -->
        <div class="absolute top-6 right-6 flex items-center">
          <button @click="playMusic" v-show="showAudio">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="36"
              height="32"
              viewBox="0 0 640 512"
            >
              <path
                d="M633.82 458.1l-69-53.33C592.42 360.8 608 309.68 608 256c0-95.33-47.73-183.58-127.65-236.03-11.17-7.33-26.18-4.24-33.51 6.95-7.34 11.17-4.22 26.18 6.95 33.51 66.27 43.49 105.82 116.6 105.82 195.58 0 42.78-11.96 83.59-33.22 119.06l-38.12-29.46C503.49 318.68 512 288.06 512 256c0-63.09-32.06-122.09-85.77-156.16-11.19-7.09-26.03-3.8-33.12 7.41-7.09 11.2-3.78 26.03 7.41 33.13C440.27 165.59 464 209.44 464 256c0 21.21-5.03 41.57-14.2 59.88l-39.56-30.58c3.38-9.35 5.76-19.07 5.76-29.3 0-31.88-17.53-61.33-45.77-76.88-11.58-6.33-26.19-2.16-32.61 9.45-6.39 11.61-2.16 26.2 9.45 32.61 11.76 6.46 19.12 18.18 20.4 31.06L288 190.82V88.02c0-21.46-25.96-31.98-40.97-16.97l-49.71 49.7L45.47 3.37C38.49-2.05 28.43-.8 23.01 6.18L3.37 31.45C-2.05 38.42-.8 48.47 6.18 53.9l588.36 454.73c6.98 5.43 17.03 4.17 22.46-2.81l19.64-25.27c5.41-6.97 4.16-17.02-2.82-22.45zM32 184v144c0 13.25 10.74 24 24 24h102.06l88.97 88.95c15.03 15.03 40.97 4.47 40.97-16.97V352.6L43.76 163.84C36.86 168.05 32 175.32 32 184z"
                fill="#000000"
              />
            </svg>
          </button>
          <button @click="stopMusic" v-show="!showAudio">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="36"
              height="32"
              viewBox="0 0 576 512"
            >
              <path
                fill="#000000"
                d="M215.03 71.05L126.06 160H24c-13.26 0-24 10.74-24 24v144c0 13.25 10.74 24 24 24h102.06l88.97 88.95c15.03 15.03 40.97 4.47 40.97-16.97V88.02c0-21.46-25.96-31.98-40.97-16.97m233.32-51.08c-11.17-7.33-26.18-4.24-33.51 6.95c-7.34 11.17-4.22 26.18 6.95 33.51c66.27 43.49 105.82 116.6 105.82 195.58S488.06 408.1 421.79 451.59c-11.17 7.32-14.29 22.34-6.95 33.5c7.04 10.71 21.93 14.56 33.51 6.95C528.27 439.58 576 351.33 576 256S528.27 72.43 448.35 19.97M480 256c0-63.53-32.06-121.94-85.77-156.24c-11.19-7.14-26.03-3.82-33.12 7.46s-3.78 26.21 7.41 33.36C408.27 165.97 432 209.11 432 256s-23.73 90.03-63.48 115.42c-11.19 7.14-14.5 22.07-7.41 33.36c6.51 10.36 21.12 15.14 33.12 7.46C447.94 377.94 480 319.54 480 256m-141.77-76.87c-11.58-6.33-26.19-2.16-32.61 9.45c-6.39 11.61-2.16 26.2 9.45 32.61C327.98 228.28 336 241.63 336 256c0 14.38-8.02 27.72-20.92 34.81c-11.61 6.41-15.84 21-9.45 32.61c6.43 11.66 21.05 15.8 32.61 9.45c28.23-15.55 45.77-45 45.77-76.88s-17.54-61.32-45.78-76.86"
              />
            </svg>
          </button>
        </div>

        <!-- Header -->
        <div class="flex flex-row justify-center mt-6">
          <img class="h-48 w-48" src="../src/assets/img/bingo.png" />
        </div>
        <div class="breadcrumbs text-xl border px-5 border-black rounded-md">
          <ul>
            <li class="font-normal">MODE</li>
            <li class="font-semibold">{{ level.toUpperCase() }}</li>
            <li :style="{ color: numbers.length === 50 ? 'black' : 'red' }">
              {{ numbers.length }} Balls Left!
            </li>
          </ul>
        </div>

        <div class="flex flex-row items-center mt-6">
          <div
            class="p-3 shadow-md rounded-full w-16 h-16 text-center border border-neutral-950 border-r-4"
          >
            <p class="text-3xl font-bold text-black">
              {{ usedNumber[usedNumber.length - 1] }}
            </p>
          </div>
        </div>

        <div class="flex flex-row justify-center m-8">
          <button
            class="btn mr-3"
            :disabled="
              randomBtnText === 'Out Of Number!' || toDisabledwhileRandom
            "
            @click="startAutoRandomNumber"
          >
            {{ randomBtnText }}
          </button>
          <button
            class="btn btn-warning"
            @click="shuffleNumber"
            :disabled="toDisabledwhileRandom"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="32"
              height="32"
              viewBox="0 0 24 24"
            >
              <g
                fill="none"
                stroke="#ffffff"
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="1.5"
                color="#ffffff"
              >
                <path
                  d="M20.5 5.5h-11C5.787 5.5 3 8.185 3 12m.5 6.5h11c3.713 0 6.5-2.685 6.5-6.5"
                />
                <path
                  d="M18.5 3S21 4.841 21 5.5S18.5 8 18.5 8m-13 8S3 17.841 3 18.5S5.5 21 5.5 21"
                />
              </g>
            </svg>
          </button>
        </div>

        <div class="flex flex-col items-center mt-8">
          <!-- Display the first 5 numbers -->
          <div
            class="relative p-6 border-4 border-blue-500 rounded-md bg-white shadow-xl flex items-center justify-center w-auto min-w-max"
          >
            <div class="flex flex-row gap-4">
              <div
                v-for="(num, index) in visibleNumbers.toReversed()"
                :key="index"
                class="flex w-16 h-16 items-center justify-center rounded-full shadow-lg bg-gradient-to-r from-purple-400 via-pink-500 to-red-50"
              >
                <p class="text-2xl font-bold text-white">
                  {{ num }}
                </p>
              </div>
            </div>
          </div>

          <!-- Hidden nums -->
          <div @click="showHiddenNumbers = !showHiddenNumbers">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="32"
              height="32"
              viewBox="0 0 512 512"
            >
              <path
                d="M98.9 184.7l1.8 2.1 136 156.5c4.6 5.3 11.5 8.6 19.2 8.6 7.7 0 14.6-3.4 19.2-8.6L411 187.1l2.3-2.6c1.7-2.5 2.7-5.5 2.7-8.7 0-8.7-7.4-15.8-16.6-15.8H112.6c-9.2 0-16.6 7.1-16.6 15.8 0 3.3 1.1 6.4 2.9 8.9z"
                fill="#d9d9d9"
              />
            </svg>
          </div>

          <!-- Display hidden numbers -->
          <div v-if="showHiddenNumbers" class="mt-4 p-4 bg-gray-100 rounded-lg">
            <div class="flex flex-wrap gap-2 justify-center">
              <div
                v-for="(num, index) in hiddenNumbers.toReversed()"
                :key="index"
                class="flex w-12 h-12 items-center justify-center rounded-full shadow-lg bg-gradient-to-r from-green-400 to-blue-50"
              >
                <p class="text-xl font-bold text-black">
                  {{ num }}
                </p>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Countdown Popup -->
      <div
        v-show="alertCountdown"
        class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50"
      >
        <div class="bg-white p-8 rounded-lg shadow-lg text-center">
          <p class="text-3xl font-bold">Starting in {{ countdown }} seconds</p>
        </div>
      </div>

      <!-- Bingo Board -->
      <div class="w-1/2 flex justify-center items-center mt-24">
        <div class="w-4/5">
          <table
            class="jersey-20-regular table-lg bg-white m-9 rounded-lg table-zebra"
          >
            <!-- head -->
            <thead>
              <tr>
                <th class="bg-red-500 text-white text-3xl">B</th>
                <th class="bg-yellow-500 text-white text-3xl">I</th>
                <th class="bg-green-500 text-white text-3xl">N</th>
                <th class="bg-blue-500 text-white text-3xl">G</th>
                <th class="bg-purple-500 text-white text-3xl">O</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="i in 5" :key="i">
                <td
                  v-for="j in 5"
                  :key="j"
                  :id="shuffledNumbers[(i - 1) * 5 + (j - 1)]?.toString()"
                  @click="
                    toggleSelection(shuffledNumbers[(i - 1) * 5 + (j - 1)])
                  "
                  :class="[
                    'h-20 w-20 text-center cursor-pointer',
                    isSelected(shuffledNumbers[(i - 1) * 5 + (j - 1)])
                      ? `bg-pink-500 text-white`
                      : '',
                    shuffledNumbers[(i - 1) * 5 + (j - 1)] ===
                      usedNumber[usedNumber.length - 1]
                  ]"
                >
                  {{ shuffledNumbers[(i - 1) * 5 + (j - 1)] }}
                </td>
              </tr>
            </tbody>
          </table>

          <!-- Alert BG-->
          <div
            v-show="hasWon"
            class="fixed inset-0 bg-gray-400 bg-opacity-40 flex items-center justify-center"
          >
            <!-- Alert -->
            <div
              class="bounce-in-top relative card card-side bg-base-100 shadow-xl w-96 overflow-hidden"
            >
              <!-- Video Background -->
              <video
                class="absolute inset-0 w-full h-full object-cover"
                autoplay
                loop
                muted
              >
                <source src="/src/assets/video/heart.mp4" type="video/mp4" />
                Your browser does not support the video tag.
              </video>

              <!-- Content over Video -->
              <div class="relative z-10 card-body text-white">
                <h2 class="card-title">Awesome!</h2>
                <p class="">You’re the bingo winner!</p>
                <div class="card-actions justify-end">
                  <button
                    @click="resetGame"
                    class="btn bg-yellow-400 text-white"
                  >
                    Play again
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.bounce-in-top {
  -webkit-animation: bounce-in-top 1.1s both;
  animation: bounce-in-top 1.1s both;
}

@-webkit-keyframes bounce-in-top {
  0% {
    -webkit-transform: translateY(-500px);
    transform: translateY(-500px);
    -webkit-animation-timing-function: ease-in;
    animation-timing-function: ease-in;
    opacity: 0;
  }
  38% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    -webkit-animation-timing-function: ease-out;
    animation-timing-function: ease-out;
    opacity: 1;
  }
  55% {
    -webkit-transform: translateY(-65px);
    transform: translateY(-65px);
    -webkit-animation-timing-function: ease-in;
    animation-timing-function: ease-in;
  }
  72% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    -webkit-animation-timing-function: ease-out;
    animation-timing-function: ease-out;
  }
  81% {
    -webkit-transform: translateY(-28px);
    transform: translateY(-28px);
    -webkit-animation-timing-function: ease-in;
    animation-timing-function: ease-in;
  }
  90% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    -webkit-animation-timing-function: ease-out;
    animation-timing-function: ease-out;
  }
  95% {
    -webkit-transform: translateY(-8px);
    transform: translateY(-8px);
    -webkit-animation-timing-function: ease-in;
    animation-timing-function: ease-in;
  }
  100% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    -webkit-animation-timing-function: ease-out;
    animation-timing-function: ease-out;
  }
}
@keyframes bounce-in-top {
  0% {
    -webkit-transform: translateY(-500px);
    transform: translateY(-500px);
    -webkit-animation-timing-function: ease-in;
    animation-timing-function: ease-in;
    opacity: 0;
  }
  38% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    -webkit-animation-timing-function: ease-out;
    animation-timing-function: ease-out;
    opacity: 1;
  }
  55% {
    -webkit-transform: translateY(-65px);
    transform: translateY(-65px);
    -webkit-animation-timing-function: ease-in;
    animation-timing-function: ease-in;
  }
  72% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    -webkit-animation-timing-function: ease-out;
    animation-timing-function: ease-out;
  }
  81% {
    -webkit-transform: translateY(-28px);
    transform: translateY(-28px);
    -webkit-animation-timing-function: ease-in;
    animation-timing-function: ease-in;
  }
  90% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    -webkit-animation-timing-function: ease-out;
    animation-timing-function: ease-out;
  }
  95% {
    -webkit-transform: translateY(-8px);
    transform: translateY(-8px);
    -webkit-animation-timing-function: ease-in;
    animation-timing-function: ease-in;
  }
  100% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    -webkit-animation-timing-function: ease-out;
    animation-timing-function: ease-out;
  }
}

.jersey-20-regular {
  font-family: "Jersey 20", sans-serif;
  font-weight: 400;
  font-style: normal;
}

.bg-customRed {
  background-color: #f24452;
}

.button {
  position: relative;
  display: inline-block;
  margin: 20px;
}

.button a {
  color: white;
  font-family: Helvetica, sans-serif;
  font-weight: bold;
  font-size: 36px;
  text-align: center;
  text-decoration: none;
  background-color: #e74c3c; /* สีแดงหลัก */
  display: block;
  position: relative;
  padding: 20px 40px;

  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  text-shadow: 0px 1px 0px #000;
  filter: dropshadow(color=#000, offx=0px, offy=1px);

  -webkit-box-shadow: inset 0 1px 0 #ffccc7, 0 10px 0 #8b0000;
  -moz-box-shadow: inset 0 1px 0 #ffccc7, 0 10px 0 #8b0000;
  box-shadow: inset 0 1px 0 #ffccc7, 0 10px 0 #8b0000;

  -webkit-border-radius: 5px;
  -moz-border-radius: 5px;
  border-radius: 5px;
}

.button a:active {
  top: 10px;
  background-color: #c0392b;

  -webkit-box-shadow: inset 0 1px 0 #ffccc7, inset 0 -3px 0 #8b0000;
  -moz-box-shadow: inset 0 1px 0 #ffccc7, inset 0 -3px 0 #8b0000;
  box-shadow: inset 0 1px 0 #ffccc7, inset 0 -3px 0 #8b0000;
}

.button:after {
  content: "";
  height: 100%;
  width: 100%;
  padding: 4px;
  position: absolute;
  bottom: -15px;
  left: -4px;
  z-index: -1;
  background-color: #5e1d1a; /* เงาสีแดงเข้ม */
  -webkit-border-radius: 5px;
  -moz-border-radius: 5px;
  border-radius: 5px;
}

.button a:hover {
  background-color: #b81212;
  color: #fff;
  -webkit-box-shadow: inset 0 1px 0 #ffccc7, 0 8px 0 #8b0000;
  -moz-box-shadow: inset 0 1px 0 #ffccc7, 0 8px 0 #8b0000;
  box-shadow: inset 0 1px 0 #ffccc7, 0 8px 0 #8b0000;
  text-shadow: 0px 1px 0px #333;
}

.text-focus-in {
  -webkit-animation: text-focus-in 0.7s cubic-bezier(0.55, 0.085, 0.68, 0.53)
    both;
  animation: text-focus-in 0.7s cubic-bezier(0.55, 0.085, 0.68, 0.53) both;
}
@-webkit-keyframes text-focus-in {
  0% {
    -webkit-filter: blur(12px);
    filter: blur(12px);
    opacity: 0;
  }
  100% {
    -webkit-filter: blur(0px);
    filter: blur(0px);
    opacity: 1;
  }
}
@keyframes text-focus-in {
  0% {
    -webkit-filter: blur(12px);
    filter: blur(12px);
    opacity: 0;
  }
  100% {
    -webkit-filter: blur(0px);
    filter: blur(0px);
    opacity: 1;
  }
}
</style>
