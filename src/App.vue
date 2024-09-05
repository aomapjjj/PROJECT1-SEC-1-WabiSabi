<script setup>
import { ref, onMounted, computed, watch } from "vue"

const numbers = ref(Array.from(Array(76).keys()).splice(1))
const drawnNumbers = ref([])
const usedNumber = ref([])
const randomBtnText = ref("Start Bingo Game")
const toDisabledwhileRandom = ref(false)
const shuffledNumbers = ref([])
const selectedNumbers = ref([])
const level = ref("default")
const gameStart = ref(false)
const countdown = ref(4)
const alertCountdown = ref(false)
const showCard1 = ref(false)
const showCard2 = ref(false)
const showWhileRandom = ref(false)
const numberWhileRandom = ref()

// 1-75 bingo table
const allNumbers = ref(Array.from(Array(76).keys()).splice(1))
const bingoTable = ref([[], [], [], [], []])
const highlightedNumbers = ref([])

let autoRandomInterval = null

const setLevel = (newLevel) => {
  gameStart.value = true
  level.value = newLevel
}

onMounted(() => {
  shuffleNumber()
})

// Randomly select 36 unique numbers
const select36Numbers = () => {
  while (drawnNumbers.value.length < 35) {
    let randomIndex = Math.floor(Math.random() * numbers.value.length)
    let number = numbers.value.splice(randomIndex, 1)[0]
    drawnNumbers.value.push(number)
  }
}

select36Numbers()

const randomNumber = () => {
  let randomIndex = Math.floor(Math.random() * drawnNumbers.value.length)
  let number = drawnNumbers.value.splice(randomIndex, 1)[0]
  usedNumber.value.push(number)
  numberWhileRandom.value = number

  if (drawnNumbers.value.length === 0) {
    randomBtnText.value = "Out Of Number!"
    clearInterval(autoRandomInterval)
    showAlertLose.value = true
    loseSoundPlay()
  }
  toDisabledwhileRandom.value = true
}

watch(numberWhileRandom, (newValue) => {
  if (newValue !== null) {
    highlightedNumbers.value.push(newValue)
  }
})

const isHighlighted = (number) => {
  return highlightedNumbers.value.includes(number)
}

// Generate Bingo Table - ลูป 5 ครั้งตาม col B,I,N,G,O แต่ละรอบการวน จะดึงตัวเลข 15 ตัวแรกจาก numbers
// splice ช่วยลบเลขถัดไป เช่น 1-15 แล้วไป 16-30 อะ แล้วมันก็ช่วย sort เลขด้วย
const randomNumsinBorad = [...allNumbers.value]

const generateBingoTable = () => {
  for (let i = 0; i < 5; i++) {
    bingoTable.value[i] = randomNumsinBorad.splice(0, 15)
  }
  return bingoTable.value
}
console.log(generateBingoTable())

let numberOnBoard = Array.apply(null, { length: 76 }).map(Number.call, Number)
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

//console.log(numberOnBoard)

const toggleSelection = (number) => {
  if (usedNumber.value.includes(number)) {
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

//console.log("selectedNumbers.value", selectedNumbers.value)

const musicPlayer = ref("")
const playingMusic = ref(false)

const onOffMusic = () => {
  playingMusic.value = !playingMusic.value
  if (playingMusic.value) musicPlayer.value.play()
  else musicPlayer.value.pause()
  console.log(playingMusic.value)
}

const clicksound = ref("")

const clickMusic = () => {
  clicksound.value.play()
}

const loseSoundEffect = ref("")

const loseSoundPlay = () => {
  loseSoundEffect.value.play()
}

const winSoundEffect = ref("")

const winSoundPlay = () => {
  winSoundEffect.value.play()
}

const visibleNumbers = computed(() => {
  return usedNumber.value.slice(-5)
})

console.log("visibleNumbers" + usedNumber.value)

//console.log(visibleNumbers)

const checkLineWin = () => {

  // Check rows
  for (let row = 0; row < 5; row++) {
    // อันนี้คือเช็คแต่ละ row นะ มันเลยสามารถกด 5 ตัวโดยที่ไม่สนกันได้
    let allMarked = true
    // console.log(`Checking row ${row}`)

    for (let col = 0; col < 5; col++) {
      // อันนี้คือเช็คแต่ละ column ของ row เช่น index ที่ 5 ก็จะเป็น row 1 column 0
      const index = row * 5 + col // อันนี้คือ อินเด้กของมัน
      if (!selectedNumbers.value.includes(shuffledNumbers.value[index])) {
        // มันจะทำงานโดยการเช็คตัวที่ไม่ได้ มาค ไม่ได้เช็คตัวที่มาคนะ
        allMarked = false
        
        // console.log(
        //   `Row ${row}, Col ${col} is not marked. Value: ${shuffledNumbers.value[index]}`
        // )
      }
    }
    if (allMarked) {
      console.log(`Row ${row} is completely marked.`)
      return true
    }


  }

  // Check columns
  for (let col = 0; col < 5; col++) {
    // อารมเดียวกันกะข้างบน
    let allMarked = true
    for (let row = 0; row < 5; row++) {
      const index = row * 5 + col
      if (!selectedNumbers.value.includes(shuffledNumbers.value[index])) {
        allMarked = false
    
      }
    }
    if (allMarked) return true
  }

  // Check diagonals
  let allMarked = true
  for (let i = 0; i < 5; i++) {
    //อันนี้หลักการเดียวกัน แต่อันนี้จะวนอาเรย์
    const index = i * 5 + i //ถ้าลองคิดดูแนวทะแยงจะมีแค่ index 0 6 12 18 25 เหมือนคิดเลขนั่นแหละ
    if (!selectedNumbers.value.includes(shuffledNumbers.value[index])) {
      //อันนี้เช็คว่าไม่ใช่อินเด้กที่เราตั้งไว้ใช่มั้ย
      allMarked = false
  
    }
  }
  if (allMarked) return true

  // Check Anti-diagonal
  allMarked = true
  for (let i = 0; i < 5; i++) {
    // เมิลกัล
    const index = i * 5 + (4 - i)
    if (!selectedNumbers.value.includes(shuffledNumbers.value[index])) {
      allMarked = false
  
    }
  }
  if (allMarked) return true
  return false
}

const checkBlackoutWin = () => {
  winSoundPlay()
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
  }, 5000)

  alertCountdown.value = true
  startCountdown()
  showWhileRandom.value = true
}

const startCountdown = () => {
  let interval = setInterval(() => {
    countdown.value--
    if (countdown.value <= 0) {
      clearInterval(interval)
      alertCountdown.value = false
    }
  }, 2000)
}

const showAlertLose = ref(false)
const showAlertWin = ref(false)

const handleBingoClick = () => {
  if (hasWon.value) {
    winSoundPlay()
    showAlertWin.value = true

  }
}

// ฟังก์ชันที่ใช้รีเซ็ตเกม
const resetGame = () => {
  showAlertWin.value = false
  showAlertLose.value = false
  hasWon.value = false
  gameStart.value = false
  usedNumber.value = []
  selectedNumbers.value = []
  shuffleNumber()
  randomBtnText.value = "Start Bingo Game"
  numbers.value = Array.from(Array(76).keys()).splice(1)
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
      <source src="/video/bgbingo.mp4" type="video/mp4" />
    </video>

    <!-- Level Selection -->
    <div class="absolute inset-0 flex flex-col mt-32 ml-4" v-if="!gameStart">
      <div class="text-focus-in">
        <div class="flex items-center justify-between">
          <div class="">
            <div class="text-7xl m-5 flex items-center">
              <p>
                Dive into the Fun <br />with
                <span class="text-red-500 jersey-20-regular">B</span>
                <span class="text-yellow-500 jersey-20-regular">I</span>
                <span class="text-green-500 jersey-20-regular">N</span>
                <span class="text-blue-500 jersey-20-regular">G</span>
                <span class="text-purple-500 jersey-20-regular">O</span>
                <span> Game</span>
              </p>
              <svg
                class="mt-20"
                xmlns="http://www.w3.org/2000/svg"
                width="64"
                height="64"
                viewBox="0 0 48 48"
              >
                <path
                  fill="#45413c"
                  d="M15.52 44.18a8.48 1.82 0 1 0 16.96 0a8.48 1.82 0 1 0-16.96 0"
                  opacity=".15"
                />
                <path
                  fill="#ff6242"
                  d="M25.4 2.5h-2.8c-1.86 0-3.34 1.18-3.23 2.57L21 26.32c.09 1.18 1.4 2.11 3 2.11s2.88-.93 3-2.11l1.63-21.25c.11-1.39-1.37-2.57-3.23-2.57"
                />
                <path
                  fill="#ff866e"
                  d="M19.56 7.48a3.31 3.31 0 0 1 3-1.6h2.8a3.31 3.31 0 0 1 3 1.6l.19-2.41c.11-1.39-1.37-2.57-3.23-2.57H22.6c-1.86 0-3.34 1.18-3.23 2.57Z"
                />
                <path
                  fill="none"
                  stroke="#45413c"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M25.4 2.5h-2.8c-1.86 0-3.34 1.18-3.23 2.57L21 26.32c.09 1.18 1.4 2.11 3 2.11s2.88-.93 3-2.11l1.63-21.25c.11-1.39-1.37-2.57-3.23-2.57"
                />
                <path
                  fill="#ff6242"
                  d="M20.35 35.24a3.65 3.65 0 1 0 7.3 0a3.65 3.65 0 1 0-7.3 0"
                />
                <path
                  fill="#ff866e"
                  d="M24 33.93A3.58 3.58 0 0 1 27.57 36a4 4 0 0 0 .08-.77a3.65 3.65 0 1 0-7.3 0a4 4 0 0 0 .08.77A3.58 3.58 0 0 1 24 33.93"
                />
                <path
                  fill="none"
                  stroke="#45413c"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M20.35 35.24a3.65 3.65 0 1 0 7.3 0a3.65 3.65 0 1 0-7.3 0"
                />
              </svg>
            </div>
            <div class="flex items-center space-x-4 mt-5 ml-5 px-4 py-3">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="32"
                height="32"
                viewBox="0 0 48 48"
              >
                <path
                  fill="#ffe500"
                  d="M4 21.5a20 20 0 1 0 40 0a20 20 0 1 0-40 0"
                />
                <path
                  fill="#ebcb00"
                  d="M24 1.5a20 20 0 1 0 20 20a20 20 0 0 0-20-20m0 37a18.25 18.25 0 1 1 18.25-18.25A18.25 18.25 0 0 1 24 38.5"
                />
                <path
                  fill="#fff48c"
                  d="M18 5.5a6 1.5 0 1 0 12 0a6 1.5 0 1 0-12 0"
                />
                <path
                  fill="#45413c"
                  d="M8 45.5a16 1.5 0 1 0 32 0a16 1.5 0 1 0-32 0"
                  opacity=".15"
                />
                <path
                  fill="none"
                  stroke="#45413c"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M4 21.5a20 20 0 1 0 40 0a20 20 0 1 0-40 0"
                />
                <path
                  fill="#ffaa54"
                  d="M38.5 26.5c0 .83-1.12 1.5-2.5 1.5s-2.5-.67-2.5-1.5S34.62 25 36 25s2.5.67 2.5 1.5m-29 0c0 .83 1.12 1.5 2.5 1.5s2.5-.67 2.5-1.5S13.38 25 12 25s-2.5.67-2.5 1.5"
                />
                <path
                  fill="#ffb0ca"
                  stroke="#45413c"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M17.94 29.5a.94.94 0 0 0-.71.33a1 1 0 0 0-.22.76a7.09 7.09 0 0 0 14 0a1 1 0 0 0-.22-.76a.94.94 0 0 0-.71-.33Z"
                />
                <path
                  fill="#ff87af"
                  stroke="#45413c"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M29.57 33.84A11.4 11.4 0 0 0 24 32.53a11.4 11.4 0 0 0-5.57 1.31a7.16 7.16 0 0 0 11.14 0"
                />
                <path
                  fill="none"
                  stroke="#45413c"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M10 22.75a1.75 1.75 0 0 1 3.5 0m21 0a1.75 1.75 0 0 1 3.5 0"
                />
              </svg>
              <p class="text-4xl">Choose Your Game Mode</p>
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="32"
                height="32"
                viewBox="0 0 48 48"
              >
                <path
                  fill="#ffe500"
                  d="M4 21.5a20 20 0 1 0 40 0a20 20 0 1 0-40 0"
                />
                <path
                  fill="#ebcb00"
                  d="M24 1.5a20 20 0 1 0 20 20a20 20 0 0 0-20-20m0 37a18.25 18.25 0 1 1 18.25-18.25A18.25 18.25 0 0 1 24 38.5"
                />
                <path
                  fill="#fff48c"
                  d="M18 5.5a6 1.5 0 1 0 12 0a6 1.5 0 1 0-12 0"
                />
                <path
                  fill="#45413c"
                  d="M8 45.5a16 1.5 0 1 0 32 0a16 1.5 0 1 0-32 0"
                  opacity=".15"
                />
                <path
                  fill="none"
                  stroke="#45413c"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M4 21.5a20 20 0 1 0 40 0a20 20 0 1 0-40 0"
                />
                <path
                  fill="#ffaa54"
                  d="M38.5 26.5c0 .83-1.12 1.5-2.5 1.5s-2.5-.67-2.5-1.5S34.62 25 36 25s2.5.67 2.5 1.5m-29 0c0 .83 1.12 1.5 2.5 1.5s2.5-.67 2.5-1.5S13.38 25 12 25s-2.5.67-2.5 1.5"
                />
                <path
                  fill="#ffb0ca"
                  stroke="#45413c"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M17.94 29.5a.94.94 0 0 0-.71.33a1 1 0 0 0-.22.76a7.09 7.09 0 0 0 14 0a1 1 0 0 0-.22-.76a.94.94 0 0 0-.71-.33Z"
                />
                <path
                  fill="#ff87af"
                  stroke="#45413c"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M29.57 33.84A11.4 11.4 0 0 0 24 32.53a11.4 11.4 0 0 0-5.57 1.31a7.16 7.16 0 0 0 11.14 0"
                />
                <path
                  fill="none"
                  stroke="#45413c"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M10 22.75a1.75 1.75 0 0 1 3.5 0m21 0a1.75 1.75 0 0 1 3.5 0"
                />
              </svg>
            </div>
            <div class="flex flex-row px-10 py-3">
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
            <div
              class="card bg-white w-96 shadow-xl mr-24 rounded-lg overflow-hidden border-4 border-yellow-400"
            >
              <div
                class="card-body text-center h-48 flex flex-col justify-center items-center"
              >
                <h2 class="card-title text-3xl font-bold text-yellow-400 mb-6">
                  Line Win Pattern
                </h2>
                <p class="text-lg text-green-700 font-semibold mt-2">
                  Complete a single row<br />and claim your win!
                </p>
              </div>
              <figure>
                <img
                  src="/img/bingoWinLine.png"
                  alt="Line"
                  class="w-56 h-56 object-contain"
                />
              </figure>
            </div>
          </div>

          <div class="flex justify-end" v-if="showCard2">
            <div
              class="card bg-white w-96 shadow-xl mr-24 rounded-lg overflow-hidden border-4 border-blue-400"
            >
              <div
                class="card-body text-center h-48 flex flex-col justify-center items-center"
              >
                <h2 class="card-title text-3xl font-bold text-blue-400 mb-6">
                  Blackout Win Pattern
                </h2>
                <p class="text-lg text-green-700 font-semibold mt-2">
                  Fill the entire card to<br />achieve ultimate victory!
                </p>
              </div>
              <figure>
                <img
                  class="w-56 h-56 object-contain"
                  src="/img/bingoWinBlackout.png"
                  alt="Blackout"
                />
              </figure>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- game content -->
    <div v-if="gameStart" class="relative z-10 flex flex-row w-full h-full">
      <!-- Generate Bingo Table -->
      <div class="w-1/4 flex flex-col justify-center items-center mt-10 ml-16">
        <table
          class="table-sm bg-white m-1 rounded-lg table-zebra border border-black"
        >
          <thead>
            <tr>
              <th class="bg-red-500 text-white text-lg border border-black">
                B
              </th>
              <th class="bg-yellow-500 text-white text-lg border border-black">
                I
              </th>
              <th class="bg-green-500 text-white text-lg border border-black">
                N
              </th>
              <th class="bg-blue-500 text-white text-lg border border-black">
                G
              </th>
              <th class="bg-purple-500 text-white text-lg border border-black">
                O
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="row in 15" :key="row" class="border border-black">
              <td
                v-for="col in 5"
                :key="col"
                class="border border-black"
                :class="{
                  'highlighted-cell': isHighlighted(
                    bingoTable[col - 1][row - 1]
                  )
                }"
              >
                {{ bingoTable[col - 1][row - 1] }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="w-1/2 flex flex-col justify-center items-center">
        <!-- Audio Controls -->
        <div class="absolute top-6 right-6 flex items-center">
          <label class="swap">
            <!-- this hidden checkbox controls the state -->
            <input type="checkbox" />

            <audio controls ref="musicPlayer" class="hidden" loop>
              <source src="/audio/bgmusic1.mp3" type="audio/mp3" />
            </audio>

            <audio controls ref="clicksound" class="hidden">
              <source src="/audio/click-sound1.wav" type="audio/wav" />
            </audio>

            <!-- volume on icon -->
            <svg
              @click="onOffMusic"
              class="swap-on fill-current"
              xmlns="http://www.w3.org/2000/svg"
              width="48"
              height="48"
              viewBox="0 0 24 24"
            >
              <path
                d="M14,3.23V5.29C16.89,6.15 19,8.83 19,12C19,15.17 16.89,17.84 14,18.7V20.77C18,19.86 21,16.28 21,12C21,7.72 18,4.14 14,3.23M16.5,12C16.5,10.23 15.5,8.71 14,7.97V16C15.5,15.29 16.5,13.76 16.5,12M3,9V15H7L12,20V4L7,9H3Z"
              />
            </svg>

            <!-- volume off icon -->
            <svg
              @click="onOffMusic"
              class="swap-off fill-current"
              xmlns="http://www.w3.org/2000/svg"
              width="48"
              height="48"
              viewBox="0 0 24 24"
            >
              <path
                d="M3,9H7L12,4V20L7,15H3V9M16.59,12L14,9.41L15.41,8L18,10.59L20.59,8L22,9.41L19.41,12L22,14.59L20.59,16L18,13.41L15.41,16L14,14.59L16.59,12Z"
              />
            </svg>
          </label>
        </div>

        <!-- Header -->
        <div class="breadcrumbs text-xl border px-5 border-black rounded-md">
          <ul>
            <li class="font-normal">MODE</li>
            <li class="font-semibold">{{ level.toUpperCase() }}</li>
            <li :style="{ color: drawnNumbers.length === 35 ? 'black' : 'red' }">
              {{ drawnNumbers.length }} Balls Left!
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
          <button class="btn mr-3 btn-error">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="32"
              height="32"
              viewBox="0 0 24 24"
            >
              <path fill="#ffffff" d="M14 19h4V5h-4M6 19h4V5H6z" />
            </svg>
          </button>

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

        <div class="flex flex-col items-center mt-8" v-show="showWhileRandom">
          <!-- Display the first 5 numbers -->
          <div
            class="relative p-6 border-4 border-blue-500 rounded-full bg-white shadow-xl flex items-center justify-center w-[450px] h-[100px]"
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
      <div class="w-1/2 flex justify-center items-center">
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
                <td class="text-4xl"
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

          <!-- Add Bingo! button below the Bingo board -->
          <div class="w-full flex justify-center items-center m-2">
            <button
              class="bg-pink-500 hover:bg-pink-700 text-white font-bold py-2 px-8 rounded-full text-3xl border-2 border-white -mt-12"
              :disabled="!hasWon"
              @click="handleBingoClick"
            >
              BINGO
            </button>
          </div>

          <!-- Alert Win -->
          <div
            v-show="showAlertWin"
            class="fixed inset-0 bg-gray-400 bg-opacity-40 flex items-center justify-center"
          >

          <audio controls ref="winSoundEffect" class="hidden" >
              <source src="/audio/winsound.mp3" type="audio/mp3" />
            </audio>
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
                <source src="/video/heart.mp4" type="video/mp4" />
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

          <!-- Alert lose -->
          <div
            v-show="showAlertLose"
            class="fixed inset-0 bg-gray-400 bg-opacity-40 flex items-center justify-center"
          >
            <audio controls ref="loseSoundEffect" class="hidden" >
              <source src="/audio/losesound.mp3" type="audio/mp3" />
            </audio>

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
                <source src="/video/heart.mp4" type="video/mp4" />
                Your browser does not support the video tag.
              </video>

              <!-- Content over Video -->
              <div class="relative z-10 card-body text-white">
                <h2 class="card-title">You Lose!</h2>
                <p class="">Better luck next time</p>
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
.highlighted-cell {
  color: red;
  background-color: #fdd;
}

th {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

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
  background-color: #e74c3c;
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
