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
        <button v-if="selected !== null" @click="nextQuestion" class="next-button">
          Следващ <span class="button-arrow">→</span>
        </button>
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

// Разбъркване на масив (алгоритъм на Fisher-Yates)
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

<style scoped>
.quiz-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 2rem;
  border-radius: 12px;
  background-color: white;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* Стилове за началния екран */
.start-screen {
  text-align: center;
  padding: 2rem 0;
}

.start-screen h1 {
  color: #2c3e50;
  margin-bottom: 2rem;
  font-size: 2.5rem;
}

.start-screen p {
  font-size: 1.2rem;
  margin-bottom: 1.5rem;
  color: #34495e;
}

.question-count-options {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.reset-button,
.new-game-button {
  border: none;
  border-radius: 8px;
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s;
}

.reset-button {
  background-color: #2ecc71;
  color: white;
}

.reset-button:hover {
  background-color: #27ae60;
}

.new-game-button {
  background-color: #3498db;
  color: white;
}

.new-game-button:hover {
  background-color: #2980b9;
}

.count-option {
  padding: 0.75rem 1.5rem;
  border: 2px solid #e5e5e5;
  border-radius: 8px;
  font-size: 1.1rem;
  background-color: white;
  cursor: pointer;
  transition: all 0.2s;
}

.count-option:hover {
  border-color: #3498db;
  background-color: rgba(52, 152, 219, 0.05);
}

.count-option.selected {
  border-color: #3498db;
  background-color: #3498db;
  color: white;
}

.start-button {
  background-color: #2ecc71;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 0.85rem 2rem;
  font-size: 1.2rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s;
  margin-top: 60px;
}

.start-button:hover:not(:disabled) {
  background-color: #27ae60;
}

.start-button:disabled {
  background-color: #bdc3c7;
  cursor: not-allowed;
}

/* Стилове за активната игра */
.quiz-header {
  margin-bottom: 2rem;
  text-align: center;
}

.quiz-header h1 {
  color: #2c3e50;
  margin-bottom: 1rem;
  font-size: 2rem;
}

.progress-container {
  margin-top: 1.5rem;
}

.progress-bar {
  height: 8px;
  background-color: #f0f0f0;
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 0.5rem;
}

.progress-fill {
  height: 100%;
  background-color: #3498db;
  transition: width 0.3s ease;
}

.progress-text {
  font-size: 0.875rem;
  color: #7f8c8d;
  text-align: right;
}

.question-section h2 {
  color: #2c3e50;
  margin-bottom: 1.5rem;
  font-size: 1.5rem;
  font-weight: 600;
}

.answers-list {
  list-style: none;
  padding: 0;
}

.answers-list li {
  border: 2px solid #e5e5e5;
  padding: 0;
  margin: 0.8rem 0;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
  overflow: hidden;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.answer-content {
  padding: 1rem;
  display: flex;
  align-items: center;
  flex: 1;
}

.answer-marker {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background-color: #f0f0f0;
  color: #2c3e50;
  font-weight: bold;
  margin-right: 1rem;
  flex-shrink: 0;
  font-size: 0.875rem;
}

.answer-text {
  font-size: 1rem;
}

.answers-list li:hover:not(.disabled) {
  border-color: #3498db;
  background-color: rgba(52, 152, 219, 0.05);
}

.answers-list li.selected {
  border-color: #3498db;
}

.answers-list li.correct {
  border-color: #2ecc71;
  background-color: rgba(46, 204, 113, 0.1);
}

.answers-list li.wrong {
  border-color: #e74c3c;
  background-color: rgba(231, 76, 60, 0.1);
}

.answers-list li.disabled {
  cursor: default;
}

.answer-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  margin-right: 1rem;
  font-size: 1.2rem;
  font-weight: bold;
}

.correct-icon {
  color: #2ecc71;
}

.wrong-icon {
  color: #e74c3c;
}

.next-button {
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  margin-top: 1.5rem;
  transition: background-color 0.2s;
  display: block;
  margin-left: auto;
}

.next-button:hover {
  background-color: #2980b9;
}

.button-arrow {
  display: inline-block;
  margin-left: 0.5rem;
  transition: transform 0.2s;
}

.next-button:hover .button-arrow {
  transform: translateX(3px);
}

/* Стилове за екрана с резултати */
.results-section {
  text-align: center;
  padding: 2rem 0;
}

.results-section h2 {
  color: #2c3e50;
  margin-bottom: 2rem;
  font-size: 1.8rem;
}

.score-display {
  margin: 2rem 0;
}

.score-circle {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background-color: #f8f9fa;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 0 auto;
  border: 8px solid #3498db;
}

.score-number {
  font-size: 4rem;
  font-weight: 700;
  color: #2c3e50;
  line-height: 1;
}

.score-label {
  font-size: 1rem;
  color: #7f8c8d;
  margin-top: 0.5rem;
}

.result-message {
  font-size: 1.1rem;
  color: #2c3e50;
  margin-bottom: 2rem;
}

.final-buttons {
  display: flex;
  justify-content: center;
  gap: 1rem;
}
</style>