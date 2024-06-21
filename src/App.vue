<template>
  <div
    v-if="!getStarted"
    class="flex flex-col items-center justify-center h-screen bg-gray-100"
  >
    <h1 class="text-2xl font-bold mb-4 px-3 text-center">
      Welcome to the Quiz Application
    </h1>
    <p class="mb-6 text-lg text-gray-700">Test your knowledge and have fun!</p>
    <button
      @click="proceed"
      class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
    >
      Start Quiz
    </button>
  </div>
  <div
    v-if="getStarted"
    class="flex justify-center items-center h-screen bg-primary transition-all duration-300 ease-in-out"
  >
    <QuizQuestion />
  </div>
</template>

<script>
import QuizQuestion from "./components/QuizQuestion.vue";
export default {
  name: "App",
  components: {
    QuizQuestion,
  },
  data() {
    return {
      getStarted: false,
    };
  },
  methods: {
    proceed() {
      this.getStarted = true;
    },
  },
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
