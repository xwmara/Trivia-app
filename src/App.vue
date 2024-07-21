<script>
export default {
  name: 'TriviaApp',
  data() {
    return {
      categories: ['General Knowledge', 'Books', 'Film', 'Sports', 'Geography', 'History'],
      questionAmounts: [5, 10, 15],
      difficulties: ['easy', 'medium', 'hard'],
      selectedCategory: 'General Knowledge',
      selectedAmount: 5,
      selectedDifficulty: 'easy',
      categoryMapping: {
        'General Knowledge': 9,
        'Books': 10,
        'Film': 11,
        'Sports': 21,
        'Geography': 22,
        'History': 23
      },
      quizStarted: false,
      questions: [],
      currentQuestionIndex: 0,
      score: 0,
      moneyEarned: 0
    };
  },
  computed: {
    shuffledAnswers() {
      if (this.questions.length > 0) {
        const currentQuestion = this.questions[this.currentQuestionIndex];
        return this.shuffleAnswers([
          ...currentQuestion.incorrect_answers,
          currentQuestion.correct_answer
        ]);
      }
      return [];
    },
    canUnlockMoreQuestions() {
      return this.moneyEarned >= 100;
    }
  },
  methods: {
    selectCategory(category) {
      this.selectedCategory = category;
    },
    selectAmount(amount) {
      this.selectedAmount = amount;
    },
    selectDifficulty(difficulty) {
      this.selectedDifficulty = difficulty;
    },
    async startQuiz() {
      if (!this.selectedCategory || !this.selectedDifficulty) {
        alert('Please select category and difficulty.');
        return;
      }

      try {
        const response = await fetch(`https://opentdb.com/api.php?amount=${this.selectedAmount}&category=${this.categoryMapping[this.selectedCategory]}&difficulty=${this.selectedDifficulty.toLowerCase()}&type=multiple`);
        const data = await response.json();
        this.questions = data.results;
        this.quizStarted = true;
      } catch (error) {
        console.error('Error fetching quiz data:', error);
      }
    },
    checkAnswer(answer) {
      const currentQuestion = this.questions[this.currentQuestionIndex];
      if (answer === currentQuestion.correct_answer) {
        this.score++;
        this.moneyEarned += 100;
      }
      this.currentQuestionIndex++;
    },
    shuffleAnswers(answers) {
      return answers.sort(() => Math.random() - 0.5);
    },
    restartQuiz() {
      this.quizStarted = false;
      this.questions = [];
      this.currentQuestionIndex = 0;
      this.score = 0;
      this.moneyEarned = 0;
      this.selectedCategory = 'General Knowledge';
      this.selectedAmount = 5;
      this.selectedDifficulty = 'easy';
    },
    async unlockMoreQuestions() {
      if (!this.canUnlockMoreQuestions) {
        alert('You need at least $100 to unlock more questions.');
        return;
      }

      const additionalQuestionsAmount = Math.floor(this.moneyEarned / 100);

      try {
        const response = await fetch(`https://opentdb.com/api.php?amount=${additionalQuestionsAmount}&category=${this.categoryMapping[this.selectedCategory]}&difficulty=${this.selectedDifficulty.toLowerCase()}&type=multiple`);
        const data = await response.json();
        this.questions = data.results; // Display only the new questions
        this.moneyEarned -= additionalQuestionsAmount * 100;
        this.quizStarted = true;
        this.currentQuestionIndex = 0; // Reset to start with new questions
        this.score = 0; // Optionally reset score if needed
      } catch (error) {
        console.error('Error fetching additional quiz data:', error);
      }
    }
  }
};
</script>

<template>
  <div class="container">
    <h1 class="title">Quizz App</h1>

    <div v-if="!quizStarted">
      <!-- Category Selection -->
      <div class="option-group">
        <h2>Select Category</h2>
        <div v-for="category in categories" :key="category">
          <button @click="selectCategory(category)" :class="{ selected: selectedCategory === category }">{{ category }}</button>
        </div>
      </div>

      <!-- Number of Questions Selection -->
      <div class="option-group">
        <h2>Select Number of Questions</h2>
        <div v-for="amount in questionAmounts" :key="amount">
          <button @click="selectAmount(amount)" :class="{ selected: selectedAmount === amount }">{{ amount }}</button>
        </div>
      </div>

      <!-- Difficulty Selection -->
      <div class="option-group">
        <h2>Select Difficulty</h2>
        <div v-for="difficulty in difficulties" :key="difficulty">
          <button @click="selectDifficulty(difficulty)" :class="{ selected: selectedDifficulty === difficulty }">{{ difficulty }}</button>
        </div>
      </div>

      <!-- Play Button -->
      <div class="start-button">
        <button @click="startQuiz">Play</button>
      </div>
    </div>

    <div v-else>
      <!-- Quiz Questions -->
      <div v-if="currentQuestionIndex < questions.length">
        <h2 class="question">{{ questions[currentQuestionIndex].question }}</h2>
        <ul class="answers">
          <li v-for="(answer, index) in shuffledAnswers" :key="index" @click="checkAnswer(answer)">
            {{ answer }}
          </li>
        </ul>
        <p>Score: {{ score }}</p>
        <p>Money Earned: ${{ moneyEarned }}</p>
      </div>

      <!-- Quiz Results -->
      <div v-else>
        <h2>Quiz Completed!</h2>
        <p>Your final score: {{ score }} / {{ questions.length }}</p>
        <p>Total Money Earned: ${{ moneyEarned }}</p>
        <button @click="restartQuiz">Play Again</button>
        <button @click="unlockMoreQuestions" v-if="canUnlockMoreQuestions">Unlock More Questions ($100 each)</button>
        <p v-if="!canUnlockMoreQuestions">You need at least $100 to unlock more questions.</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

.title {
  font-size: 2rem;
  margin-bottom: 20px;
}

.option-group {
  margin-bottom: 20px;
}

.option-group h2 {
  font-size: 1.5rem;
  margin-bottom: 10px;
}

button {
  padding: 10px 20px;
  font-size: 1rem;
  margin: 5px;
  background-color: #3498db;
  color: white;
  border: none;
  cursor: pointer;
}

button.selected {
  background-color: #2980b9;
}

.start-button {
  margin-top: 20px;
}

.question {
  font-size: 1.5rem;
  margin-bottom: 10px;
}

.answers {
  list-style-type: none;
  padding: 0;
}

.answers li {
  margin: 10px 0;
  padding: 10px;
  background-color: #f1f1f1;
  border: 1px solid #ddd;
  cursor: pointer;
}

.answers li:hover {
  background-color: #e5e5e5;
}
</style>
