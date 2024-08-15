<script setup>
import { ref } from 'vue'
const numbers = ref([
  1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22,
  23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41,
  42, 43, 44, 45, 46, 47, 48, 49, 50
])
const usedNumber = ref([])
const randomBtnText = ref('Random Number')

const randomNumber = () => {
  let randomIndex = Math.floor(Math.random() * numbers.value.length)
  let number = numbers.value.splice(randomIndex, 1)[0]
  usedNumber.value.push(number)
  if (numbers.value.length === 0) randomBtnText.value = 'Out Of Number!'
}
</script>

<template>
  <div class="w-full h-screen">
    <div class="bg-gradient-to-r from-sky-500 to-indigo-500 w-full h-full">
      <!-- Header -->
      <div class="flex flex-row justify-center">
        <img class=" h-40 w-40" src="../src/assets/img/bingo.png">
        <!-- <img src="../src/assets/img/logo.png"> -->
        <!-- <h1 class="text-3xl text-white font-bold m-10">Bingo Game</h1> -->
      </div>

      <div class="flex flex-col items-center">
        <div class="bg-white p-4 rounded-full shadow-lg w-6/12 text-center">
          <p class="text-3xl font-bold text-indigo-500">
            {{ usedNumber[usedNumber.length - 1] || 'No numbers selected yet' }}
          </p>
        </div>
      </div>

      <div class="flex flex-row justify-center mt-8">
        <button
          class="btn"
          :disabled="randomBtnText === 'Out Of Number!'"
          @click="randomNumber"
        >
          {{ randomBtnText }}
        </button>
      </div>

      <!-- Random Numbers Display -->
      <div class="flex flex-col items-center mt-8">
        <div class="flex flex-wrap gap-4 justify-center">
          <div
            v-for="(num, index) in usedNumber.toReversed()"
            :key="index"
            class="flex w-16 h-16 items-center justify-center rounded-full shadow-lg bg-white"
          >
            <p class="text-2xl font-bold text-indigo-500">
              {{ num }}
            </p>
          </div>
        </div>
      </div>

      
    </div>
  </div>
</template>

<style scoped></style>
