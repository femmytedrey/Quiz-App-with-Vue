<template>
  <div
    v-if="notSubmitted"
    class="bg-secondary w-[90%] md:max-w-[500px] h-fit rounded-xl shadow-lg flex flex-col overflow-hidden"
  >
    <div
      class="px-5 py-3 flex flex-col sm:flex-row justify-between items-start md:items-center shadow-md gap-y-2"
    >
      <p class="font-bold text-xl">Awesome Quiz Application</p>
      <div class="bg-[#a1c4e9] flex gap-x-2 items-center px-2 py-2 rounded-md">
        <div class="text-[#373a38]">Time Left</div>
        <div
          class="bg-[#373a38] px-2 rounded text-white font-bold"
          :class="{ 'text-red-400': dieMinute }"
        >
          {{ formatTimer }}
        </div>
      </div>
    </div>

    <!--  Display loading state -->
    <div v-if="loading" class="p-5 text-xl border-b-[1.5px]">
      Loading questions, please wait...
    </div>

    <div
      v-if="errorMsg"
      class="p-5 text-xl border-b-[1.5px] text-center font-bold"
    >
      <h1>{{ errorMsg }}</h1>
    </div>

    <div
      class="p-5 pb-8 space-y-6 border-b border-gray-300"
      v-if="questions.length"
    >
      <div class="text-xl font-bold">
        {{ next + 1 }}. {{ questions[next].question }}
      </div>
      <ul class="space-y-3">
        <li
          v-for="option in questions[next].options"
          :key="option"
          @click="SelectAnswer(option)"
          :class="[
            'w-[100%] px-4 py-2 rounded font-semibold border-[1.5px] border-[#a1c4e9]',
            lockSelection
              ? option === correctAnswer
                ? 'bg-green-300 cursor-not-allowed'
                : option === selectedAnswer
                ? 'bg-red-300 cursor-not-allowed'
                : 'cursor-not-allowed bg-blue-100'
              : 'bg-blue-100 cursor-pointer hover:bg-blue-200',
          ]"
        >
          {{ option }}
        </li>
      </ul>
    </div>
    <div class="p-5 flex justify-between items-center">
      <p class="font-bold">{{ next + 1 }} out of {{ questions.length }}</p>
      <button
        class="px-5 py-1 bg-primary text-white cursor-pointer rounded-md"
        @click="nextQuestion"
      >
        {{ btnText }}
      </button>
    </div>
  </div>

  <div
    v-else
    class="bg-secondary w-[90%] md:max-w-[500px] h-fit rounded-xl shadow-lg flex flex-col overflow-hidden space-y-5 p-6"
  >
    <div></div>
    <h1 class="text-xl text-center font-bold">
      You scored {{ score }} out of {{ questions.length }}
    </h1>
    <p class="text-xl text-center font-bold">{{ feedbackMessage }}</p>
    <div class="text-center">
      <button
        class="bg-blue-500 cursor-pointer hover:bg-blue-600 text-white px-2 py-2 rounded-2xl text-sm"
        @click="restartQuiz"
      >
        Take another quiz
      </button>
    </div>
  </div>
</template>

<script>
import { computed, onMounted, onUpdated, ref } from "vue";
export default {
  components: {},
  setup() {
    const questions = ref([]);
    const loading = ref(true);
    const next = ref(0);
    const lockSelection = ref(false);
    const selectedAnswer = ref(null);
    const score = ref(0);
    const correctAnswer = ref(null);
    const btnText = ref("Next");
    const notSubmitted = ref(true);
    const timer = ref(15 * 60);
    const intervalId = ref(null);
    const dieMinute = ref(false);
    const feedbackMessage = ref("");
    const errorMsg = ref("");

    const startTimer = () => {
      clearInterval(intervalId.value);
      intervalId.value = setInterval(() => {
        if (timer.value > 0) {
          timer.value--;
        } else {
          submitQuiz();
        }

        if (timer.value <= 30) {
          dieMinute.value = true;
        }
      }, 1000);
    };

    const formatTimer = computed(() => {
      const minutes = Math.floor(timer.value / 60);
      const seconds = timer.value % 60;
      return `${String(minutes).padStart(2, "0")}:${String(seconds).padStart(
        2,
        "0"
      )}`;
    });
    const submitQuiz = () => {
      notSubmitted.value = false;
      clearInterval(intervalId.value);
      calculateGrade();
    };

    const fetchData = async () => {
      loading.value = true;
      try {
        const data = await fetch(
          "https://femmytedrey.github.io/quiz_api/quiz.json"
        );
        if (!data.ok) {
          throw new Error("Error fetching questions...");
        }
        const fetchData = await data.json();
        const fetchedQuestions = fetchData.questions;
        questions.value = extractRandomQuiz(fetchedQuestions, 30);
      } catch (err) {
        errorMsg.value = err.message;
      } finally {
        loading.value = false;
      }
    };

    const extractRandomQuiz = (questionsArray, numQuestions) => {
      const selectedQuestion = [];
      const usedIndices = new Set();
      while (
        selectedQuestion.length < numQuestions &&
        selectedQuestion.length < questionsArray.length
      ) {
        const randomIndex = Math.floor(Math.random() * questionsArray.length);
        if (!usedIndices.has(randomIndex)) {
          selectedQuestion.push(questionsArray[randomIndex]);
          usedIndices.add(randomIndex);
        }
      }
      return selectedQuestion;
    };

    const SelectAnswer = (option) => {
      if (lockSelection.value) return;

      selectedAnswer.value = option;
      correctAnswer.value = questions.value[next.value].answer;

      if (selectedAnswer.value === correctAnswer.value) {
        score.value += 1;
      }
      lockSelection.value = true;
    };

    const nextQuestion = () => {
      if (next.value < questions.value.length - 1) {
        next.value += 1;
        lockSelection.value = false;
      } else if (btnText.value === "Submit") {
        notSubmitted.value = false;
      }
    };

    const calculateGrade = () => {
      const percentage = (score.value * questions.value.length) / 100;

      if (percentage >= 90) {
        feedbackMessage.value = "Amazing! Keep up the great work!";
      } else if (percentage >= 80) {
        feedbackMessage.value = "Great job! Well done!";
      } else if (percentage >= 70) {
        feedbackMessage.value = "Good effort! Keep practicing!";
      } else if (percentage >= 60) {
        feedbackMessage.value = "There's room for improvement. Keep trying!";
      } else {
        feedbackMessage.value = "Don't give up, study more and try again!";
      }
    };

    const restartQuiz = () => {
      next.value = 0;
      score.value = 0;
      lockSelection.value = false;
      btnText.value = "Next";
      notSubmitted.value = true;
      timer.value = 15 * 60;
      dieMinute.value = false;
      fetchData();
      startTimer();
    };

    onMounted(() => {
      fetchData();
      startTimer();
    });
    onUpdated(() => {
      if (next.value === questions.value.length - 1) {
        btnText.value = "Submit";
      }
    });
    return {
      questions,
      next,
      nextQuestion,
      SelectAnswer,
      restartQuiz,
      selectedAnswer,
      lockSelection,
      correctAnswer,
      btnText,
      notSubmitted,
      score,
      formatTimer,
      dieMinute,
      loading,
      feedbackMessage,
      errorMsg,
    };
  },
};
</script>

<style></style>
