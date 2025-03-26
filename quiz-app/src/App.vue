<template>
  <div class="app">
    <h1>Vue Quiz</h1>
    <Quiz />
  </div>
</template>

<script setup>
import Quiz from './components/Quiz.vue'
</script>

<style>
body {
  font-family: sans-serif;
  background: #f0f0f0;
  margin: 0;
}

.app {
  max-width: 600px;
  margin: 2rem auto;
  background: black;
  padding: 2rem;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
</style>