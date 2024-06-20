<template>
  <div class="flex justify-center items-center h-screen bg-primary">
    <QuizQuestion />
  </div>
</template>

<script>
import QuizQuestion from "./components/QuizQuestion.vue";
export default {
  name: "App",
  components: {
    QuizQuestion,
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
