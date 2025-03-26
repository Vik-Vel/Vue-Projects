<template>
  <div class="quiz-container">
    <!-- Начален екран с избор на брой въпроси -->
    <div v-if="!gameStarted" class="start-screen">
      <h1>Vue Quiz</h1>
      <p>Изберете брой въпроси:</p>
      <div class="question-count-options">
        <button v-for="count in [5, 10, 15, 20]" :key="count" @click="setQuestionCount(count)" class="count-option"
          :class="{ 'selected': selectedCount === count }">
          {{ count }}
        </button>
      </div>
      <button @click="startGame" class="start-button" :disabled="!selectedCount">
        Започни
      </button>
    </div>

    <!-- Екран с въпросите -->
    <div v-else-if="current < selectedQuestions.length" class="quiz-active">
      <div class="quiz-header">
        <h1>Vue Quiz</h1>
        <div class="progress-container">
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: `${(current / selectedQuestions.length) * 100}%` }"></div>
          </div>
          <div class="progress-text">Въпрос {{ current + 1 }} / {{ selectedQuestions.length }}</div>
        </div>
      </div>

      <div class="question-section">
        <h2>{{ selectedQuestions[current].question }}</h2>
        <ul class="answers-list">
          <li v-for="(answer, index) in selectedQuestions[current].answers" :key="index" @click="selectAnswer(index)"
            :class="{
              selected: selected === index,
              correct: isCorrect(index),
              wrong: isWrong(index),
              disabled: selected !== null
            }">
            <div class="answer-content">
              <span class="answer-marker">{{ ['A', 'B', 'C', 'D'][index] }}</span>
              <span class="answer-text">{{ answer }}</span>
            </div>
            <span v-if="isCorrect(index)" class="answer-icon correct-icon">✓</span>
            <span v-if="isWrong(index)" class="answer-icon wrong-icon">✗</span>
          </li>
        </ul>
        <div class="question-buttons">
          <button @click="stopGame" class="stop-button">
            Спри играта
          </button>
          <button v-if="selected !== null" @click="nextQuestion" class="next-button">
            Следващ <span class="button-arrow">→</span>
          </button>
        </div>
      </div>
    </div>

    <!-- Краен екран с резултати -->
    <div v-else class="results-section">
      <h2>Край на куиза!</h2>
      <div class="score-display">
        <div class="score-circle">
          <div class="score-number">{{ score }}</div>
          <div class="score-label">от {{ selectedQuestions.length }}</div>
        </div>
      </div>
      <p class="result-message">
        {{ getResultMessage() }}
      </p>
      <div class="final-buttons">
        <button @click="restartWithSameCount" class="reset-button">Играй със същия брой въпроси</button>
        <button @click="resetToStart" class="new-game-button">Нова игра</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { allQuestions } from './questions.js';
import '../assets/styles/quiz.css';

// Състояние на играта
const gameStarted = ref(false)
const selectedCount = ref(null)
const selectedQuestions = ref([])
const current = ref(0)
const selected = ref(null)
const score = ref(0)

// Избор на брой въпроси
const setQuestionCount = (count) => {
  selectedCount.value = count
}

// Спиране на играта и връщане към началния екран
const stopGame = () => {
  if (confirm('Сигурни ли сте, че искате да спрете играта? Прогресът ви ще бъде изгубен.')) {
    resetToStart()
  }
}

// Разбъркване на масив 
const shuffleArray = (array) => {
  const shuffled = [...array]
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
      ;[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
  }
  return shuffled
}

// Стартиране на играта с избрания брой въпроси
const startGame = () => {
  // Избор на въпроси
  const shuffledQuestions = shuffleArray(allQuestions)

  // Вземаме само броя въпроси, които потребителят е избрал
  selectedQuestions.value = shuffledQuestions.slice(0, selectedCount.value)


  // Нулиране на променливите
  current.value = 0
  selected.value = null
  score.value = 0

  // Стартиране на играта
  gameStarted.value = true
}

// Избор на отговор
const selectAnswer = (index) => {
  if (selected.value === null) {
    selected.value = index
    if (index === selectedQuestions.value[current.value].correct) {
      score.value++
    }
  }
}

// Преминаване към следващ въпрос
const nextQuestion = () => {
  selected.value = null
  current.value++
}

// Рестартиране на играта със същия брой въпроси
const restartWithSameCount = () => {
  startGame()
}

// Връщане към началния екран за нова игра
const resetToStart = () => {
  gameStarted.value = false
  selectedCount.value = null
  selectedQuestions.value = []
}

// Проверка дали избраният отговор е правилен
const isCorrect = (index) => selected.value !== null && index === selectedQuestions.value[current.value].correct

// Проверка дали избраният отговор е грешен
const isWrong = (index) => selected.value !== null && index === selected.value && index !== selectedQuestions.value[current.value].correct

// Получаване на съобщение според резултата
const getResultMessage = () => {
  const percentage = (score.value / selectedQuestions.value.length) * 100
  if (percentage === 100) return "Отлично! Перфектен резултат!"
  if (percentage >= 70) return "Добра работа! Справи се добре!"
  if (percentage >= 40) return "Не е зле, но има място за подобрение."
  return "Опитай отново, за да подобриш резултата си."
}
</script>
