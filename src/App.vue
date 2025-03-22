<template>
  <main class="flex flex-col items-center justify-center min-h-screen p-4">
    <transition name="fade">
      <div v-if="step === 'start'" key="start" class="text-center flex flex-col items-center w-[60%]">
        <h1 class="text-4xl mb-4">Welcome to the Dyslexia Reading Simulation</h1>
        <p class="text-md mb-6 text-gray-700">
          In this simulation, you will experience reading text with distortions that aim to mimic how a person with dyslexia might perceive written words.
          The goal is to help you understand the challenges they face when reading. After one minute of reading, you will answer a short quiz.
          Try your best and pay attention to how this experience feels.
        </p>
        <button @click="startReading" class="bg-blue-500 text-white px-6 py-2 rounded-xl shadow-md">Start Simulation</button>
      </div>
    </transition>

    <transition name="fade">
      <div v-if="step === 'reading'" key="reading" class="text-center max-w-xl">
        <div class="sticky top-0 bg-white p-2">
          <h2 class="text-2xl mb-2">Reading Time!</h2>
          <p class="text-red-500 font-bold mb-2">Time left: {{ timer }}s</p>
        </div>
        <div class="overflow-y-auto h-[70vh] p-4 bg-gray-50 rounded-2xl shadow-inner">
          <p class="whitespace-pre-line text-justify" v-html="movingText"></p>
        </div>
      </div>
    </transition>

    <transition name="slide">
      <div v-if="step === 'quiz'" key="quiz" class="max-w-xl">
        <div class="top-0">
          <div class="flex gap-2 justify-center">
            <h2 class="text-2xl mb-4">Quiz Time!</h2>
          </div>
          <div class="flex gap-2 justify-center">
            <div v-for="(feedback, index) in score" :key="index">
              <p v-if="feedback === 1" class="mb-2">✅</p>
              <p v-if="feedback === 0" class="mb-2">❌</p>
            </div>
          </div>
          <div class="mb-6">
            <p class="font-semibold mb-2">{{ currentQuestion.q }}</p>
            <div class="flex flex-col gap-2">
              <button
                v-for="(ans, i) in currentQuestion.a"
                :key="i"
                :disabled="currentQuestion.answered"
                @click="checkAnswer(currentQuestion, ans)"
                class="px-4 py-2 rounded-xl border hover:bg-gray-200">
                {{ ans.text }}
              </button>
            </div>
            <p v-if="currentQuestion.feedback" class="mt-2" :class="currentQuestion.correct ? 'text-green-600' : 'text-red-600'">
              {{ currentQuestion.feedback }}
            </p>
          </div>
        </div>
      </div>
    </transition>

    <transition name="slide">
      <div v-if="step === 'score'" class="text-center mt-6 w-[60%]">
        <h2 class="text-3xl mb-4">Your Final Score</h2>
        <p class="text-sm mb-6 text-gray-700">
          Thank you for participating! We hope this simulation gave you insight into the reading challenges faced by people with dyslexia.
        </p>
        <div class="text-2xl mb-8 bg-white text-gray-800 w-full max-w-prose flex flex-row justify-center gap-6">
          <p class="text-green-600 text-xl mb-2">✅ Correct: {{ totalCorrect }}</p>
          <p class="text-red-600 text-xl mb-4">❌ Incorrect: {{ totalIncorrect }}</p>
        </div>
        <button @click="reset" class="bg-blue-500 text-white px-6 py-2 rounded-xl shadow">Start Over</button>
      </div>
    </transition>
  </main>
</template>

<script setup>
import { ref, computed} from 'vue'
import baseText from './text'

const READING_TIME = 60

const step = ref('start')
const currentQuestionIndex = ref(0)
const timer = ref(READING_TIME)
const score = ref([])

const movingText = ref(baseText)
let interval

const questions = ref([
  { q: 'What does the term “neurodivergent” refer to?', a: [
    { text: 'A mental disorder that needs treatment', correct: false },
    { text: 'A natural variation in how the brain works', correct: true },
    { text: 'A temporary learning difficulty', correct: false},
    { text: 'A form of physical disability', correct: false}
  ], answered: false },
  { q: 'People with dyslexia often have difficulties with:', a: [
    { text: 'Math and science', correct: false },
    { text: 'Physical coordination', correct: false },
    { text: 'Reading, writing, and spelling', correct: true },
    { text: 'Social communication', correct: false },
  ], answered: false },
  { q: 'Which of the following is NOT typically associated with ADHD?', a: [
    { text: 'Trouble focusing', correct: false },
    { text: 'High energy', correct: false },
    { text: 'Difficulty with impulse control', correct: false },
    { text: 'Strong physical coordination', correct: true }
  ], answered: false },
  { q: 'Autistic individuals often show strengths in:', a: [
    { text: 'Pattern recognition and attention to detail', correct: true },
    { text: 'Public speaking', correct: false },
    { text: 'Physical endurance', correct: false },
    { text: 'Fast reading and writing', correct: false },
  ], answered: false },
  { q: 'Dyspraxia primarily affects a person’s ability to:', a: [
    { text: 'Understand complex ideas', correct: false },
    { text: 'Physically coordinate movements', correct: true },
    { text: 'Remember facts', correct: false },
    { text: 'Communicate with others socially', correct: false }
  ], answered: false }
])

const currentQuestion = computed(() => questions.value[currentQuestionIndex.value])
const totalCorrect = computed(() => score.value.reduce((acc, curr) => acc + curr, 0))
const totalIncorrect = computed(() => questions.value.length - totalCorrect.value);

function startReading() {
  currentQuestionIndex.value = 0
  score.value = []
  step.value = 'reading'
  timer.value = READING_TIME
  interval = setInterval(() => {
    timer.value--
    if (timer.value <= 0) {
      clearInterval(interval)
      step.value = 'quiz'
    }
  }, 1000)

   // Start dyslexia simulation
  let animationInterval = setInterval(() => {
    movingText.value = scrambleText(baseText)
  }, 300)
}

function scrambleText(text) {
  return text.replace(/\w+/g, (word) => {
    if (word.length <= 3) return word
    const middle = word.slice(1, -1).split('').sort(() => Math.random() - 0.5).join('')
    return word[0] + middle + word[word.length - 1]
  })
}

function checkAnswer(question, answer) {
  if (answer.correct) {
    score.value.push(1)
  } else {
    score.value.push(0)
  }

  if(currentQuestionIndex.value < questions.value.length - 1) {
    currentQuestionIndex.value++
  } else {
    setTimeout(() => { step.value = "score" }, 300)
  }
}

function reset() {
  step.value = 'start'
  questions.value.forEach((q) => {
    q.answered = false
    q.feedback = ''
    q.correct = false
  })
}
</script>

<style>
.fade-enter-active, .fade-leave-active { transition: opacity 0.5s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
.slide-enter-active { transition: all 0.6s ease; transform: translateY(0); }
.slide-enter-from { transform: translateY(20px); opacity: 0; }
.slide-leave-active { opacity: 0; }
</style>
