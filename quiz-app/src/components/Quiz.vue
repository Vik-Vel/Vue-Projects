<template>
  <div v-if="current < questions.length">
    <h2>{{ questions[current].question }}</h2>
    <ul>
      <li v-for="(answer, index) in questions[current].answers" :key="index" @click="selectAnswer(index)"
        :class="{ selected: selected === index, correct: isCorrect(index), wrong: isWrong(index) }">
        {{ answer }}
      </li>
    </ul>
    <button v-if="selected !== null" @click="nextQuestion">Следващ</button>
  </div>

  <div v-else>
    <h2>Край! Получи {{ score }} / {{ questions.length }} точки.</h2>
    <button @click="reset">Играй отново</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const questions = [
  {
    question: "Коя е столицата на България?",
    answers: ["Пловдив", "София", "Варна"],
    correct: 1
  },
  {
    question: "Коя година е създадена България?",
    answers: ["681", "1944", "1878"],
    correct: 0
  },
  {
    question: "Кой е написал 'Под игото'?",
    answers: ["Пейо Яворов", "Христо Ботев", "Иван Вазов"],
    correct: 2
  }
]

const current = ref(0)
const selected = ref(null)
const score = ref(0)

const selectAnswer = (index) => {
  if (selected.value === null) {
    selected.value = index
    if (index === questions[current.value].correct) {
      score.value++
    }
  }
}

const nextQuestion = () => {
  selected.value = null
  current.value++
}

const reset = () => {
  current.value = 0
  selected.value = null
  score.value = 0
}

const isCorrect = (index) => selected.value !== null && index === questions[current.value].correct
const isWrong = (index) => selected.value !== null && index === selected.value && index !== questions[current.value].correct
</script>

<style scoped>
ul {
  list-style: none;
  padding: 0;
}

li {
  background: #eee;
  padding: 10px;
  margin: 8px 0;
  border-radius: 6px;
  cursor: pointer;
  transition: 0.2s;
}

li:hover {
  background: #ddd;
}

.selected {
  font-weight: bold;
}

.correct {
  background: #c8f7c5;
}

.wrong {
  background: #f7c5c5;
}

button {
  margin-top: 20px;
  padding: 10px 15px;
  font-size: 16px;
}
</style>