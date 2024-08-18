<script setup>
import { ref, onMounted, computed } from 'vue'
const numbers = ref(Array.from(Array(51).keys()).splice(1))

const usedNumber = ref([])
const randomBtnText = ref('Random Number')
const randomNumber = () => {
  let randomIndex = Math.floor(Math.random() * numbers.value.length)
  let number = numbers.value.splice(randomIndex, 1)[0]
  usedNumber.value.push(number)
  if (numbers.value.length === 0) randomBtnText.value = 'Out Of Number!'
}

let numberOnBoard = Array.apply(null, { length: 26 }).map(Number.call, Number)
numberOnBoard.shift()

const shuffledNumbers = ref([])

// Store the state of selected numbers on the bingo board
const selectedNumbers = ref([])

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
}

const toggleSelection = (number) => {
  if (number === usedNumber.value[usedNumber.value.length - 1]) {
    if (selectedNumbers.value.includes(number)) {
      selectedNumbers.value = selectedNumbers.value.filter(
        (num) => num !== number
      )
    } else {
      selectedNumbers.value.push(number)
    }
  }
}

const isSelected = (number) => selectedNumbers.value.includes(number)

// visible numbers
const visibleNumbers = computed(() => {
  return usedNumber.value.slice(-5)
})

onMounted(() => {
  shuffleNumber()
})

console.log(shuffleNumber())
</script>

<template>
  <div class="w-full h-full">
    <div class="bg-gradient-to-r from-sky-500 to-indigo-500 w-full h-full">
      <!-- Header -->
      <div class="flex flex-row justify-center">
        <img class="h-40 w-40" src="../src/assets/img/bingo.png" />
      </div>

      <div class="flex flex-col items-center">
        <div class="bg-white p-4 shadow-lg w-1/12 h-16 text-center">
          <p class="text-3xl font-bold text-indigo-500">
            {{ usedNumber[usedNumber.length - 1] }}
          </p>
        </div>
      </div>

      <div class="flex flex-row justify-center m-8">
        <button
          class="btn mr-3"
          :disabled="randomBtnText === 'Out Of Number!'"
          @click="randomNumber"
        >
          {{ randomBtnText }}
        </button>
        <button class="btn btn-warning" @click="shuffleNumber">
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

      <!-- Random Numbers Display -->
      <div class="flex flex-col items-center mt-8">
        <!-- Display the first 5 numbers -->
    <div class="relative p-6 border-4 border-indigo-500 rounded-full bg-white shadow-xl w-4/12 h-24 flex items-center justify-center">

          <div class="flex flex-wrap gap-4 justify-center">
            <div
              v-for="(num, index) in visibleNumbers.reverse()"
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

      <!-- Bingo Board -->
      <div class="overflow-x-auto flex justify-center">
        <div>
          <table class="jersey-20-regular table-lg bg-white m-9 rounded-lg table-zebra">
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
                      ? 'bg-pink-500 text-white'
                      : '',
                    shuffledNumbers[(i - 1) * 5 + (j - 1)] ===
                    usedNumber[usedNumber.length - 1]
                      ? 'hover:bg-gray-200'
                      : 'pointer-events-none opacity-50'
                  ]"
                >
                  {{ shuffledNumbers[(i - 1) * 5 + (j - 1)] }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.jersey-20-regular {
  font-family: "Jersey 20", sans-serif;
  font-weight: 400;
  font-style: normal;
}
</style>
