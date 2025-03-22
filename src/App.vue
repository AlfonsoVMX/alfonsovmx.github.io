<template>
  <main class="flex flex-col items-center justify-center min-h-screen p-4">
    <transition name="fade">
      <div v-if="step === 'start'" key="start" class="text-center">
        <h1 class="text-4xl mb-4">Ready to Read?</h1>
        <button @click="startReading" class="bg-blue-500 text-white px-6 py-2 rounded-xl shadow-md">Start Reading</button>
      </div>
    </transition>

    <transition name="fade">
      <div v-if="step === 'reading'" key="reading" class="text-center max-w-xl">
        <div class="sticky top-0 bg-white p-4 shadow-md z-10">
          <h2 class="text-2xl mb-2">Reading Time!</h2>
          <p class="text-red-500 font-bold mb-2">Time left: {{ timer }}s</p>
        </div>
        <div class="overflow-y-auto h-[70vh] p-4 bg-gray-50 rounded-2xl shadow-inner">
          <p class="whitespace-pre-line" v-html="movingText"></p>
        </div>
      </div>
    </transition>

    <transition name="slide">
      <div v-if="step === 'quiz'" key="quiz" class="max-w-xl w-full">
        <h2 class="text-2xl mb-4">Quiz Time!</h2>
        <div v-for="(question, index) in questions" :key="index" class="mb-6">
          <p class="font-semibold mb-2">{{ question.q }}</p>
          <div class="flex flex-col gap-2">
            <button
              v-for="(ans, i) in question.a"
              :key="i"
              :disabled="question.answered"
              @click="checkAnswer(index, ans)"
              class="px-4 py-2 rounded-xl border hover:bg-gray-200">
              {{ ans.text }}
            </button>
          </div>
          <p v-if="question.feedback" class="mt-2" :class="question.correct ? 'text-green-600' : 'text-red-600'">
            {{ question.feedback }}
          </p>
        </div>
        <button @click="reset" class="mt-4 bg-green-500 text-white px-6 py-2 rounded-xl shadow">Start Over</button>
      </div>
    </transition>
  </main>
</template>

<script setup>
import { ref } from 'vue'
import baseText from './text'

const step = ref('start')
const timer = ref(60)
const movingText = ref(baseText)
let interval

const questions = ref([
  { q: 'What was the reading about?', a: [{ text: 'Lorem ipsum', correct: true }, { text: 'Space travel', correct: false }], answered: false },
  { q: 'How long did you have to read?', a: [{ text: '60 seconds', correct: true }, { text: '5 minutes', correct: false }], answered: false },
  { q: 'Was it easy to understand?', a: [{ text: 'Yes', correct: true }, { text: 'No', correct: false }], answered: false },
  { q: 'What kind of text was it?', a: [{ text: 'Example text', correct: true }, { text: 'Recipe', correct: false }], answered: false },
  { q: 'Would you read more?', a: [{ text: 'Absolutely', correct: true }, { text: 'No thanks', correct: false }], answered: false }
])

function startReading() {
  step.value = 'reading'
  timer.value = 60
  interval = setInterval(() => {
    timer.value--
    if (timer.value <= 0) {
      clearInterval(interval)
      step.value = 'quiz'
    }
  }, 1000)

   // Start dyslexia simulation
  animationInterval = setInterval(() => {
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

function checkAnswer(qIndex, answer) {
  const question = questions.value[qIndex]
  if (question.answered) return
  question.answered = true
  question.correct = answer.correct
  question.feedback = answer.correct ? '✅ Correct!' : '❌ Oops, not correct.'
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
