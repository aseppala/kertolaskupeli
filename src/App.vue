<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

const GAME_DURATION = 120 // 2 minutes in seconds
const BASE_POINTS = 10
const SPEED_BONUS_THRESHOLD = 3 // seconds
const SPEED_BONUS_POINTS = 5
const STREAK_THRESHOLD = 3
const STREAK_BONUS = 5

const gameState = ref<'idle' | 'playing' | 'finished'>('idle')
const timeLeft = ref(GAME_DURATION)
const score = ref(0)
const correctAnswers = ref(0)
const currentStreak = ref(0)
const userAnswer = ref('')
const questionStartTime = ref(0)

const num1 = ref(0)
const num2 = ref(0)
const feedback = ref('')

let timerInterval: number | null = null

const correctAnswer = computed(() => num1.value * num2.value)

const formattedTime = computed(() => {
  const minutes = Math.floor(timeLeft.value / 60)
  const seconds = timeLeft.value % 60
  return `${minutes}:${seconds.toString().padStart(2, '0')}`
})

function generateQuestion() {
  num1.value = Math.floor(Math.random() * 10) + 1
  num2.value = Math.floor(Math.random() * 10) + 1
  questionStartTime.value = Date.now()
  feedback.value = ''
  userAnswer.value = ''
}

function startGame() {
  gameState.value = 'playing'
  timeLeft.value = GAME_DURATION
  score.value = 0
  correctAnswers.value = 0
  currentStreak.value = 0
  generateQuestion()
  
  timerInterval = window.setInterval(() => {
    timeLeft.value--
    if (timeLeft.value <= 0) {
      endGame()
    }
  }, 1000)
}

function endGame() {
  gameState.value = 'finished'
  if (timerInterval !== null) {
    clearInterval(timerInterval)
    timerInterval = null
  }
}

function submitAnswer() {
  if (!userAnswer.value || gameState.value !== 'playing') return
  
  const answer = parseInt(userAnswer.value)
  const responseTime = (Date.now() - questionStartTime.value) / 1000
  
  if (answer === correctAnswer.value) {
    let points = BASE_POINTS
    
    // Speed bonus
    if (responseTime < SPEED_BONUS_THRESHOLD) {
      points += SPEED_BONUS_POINTS
    }
    
    // Update streak
    currentStreak.value++
    
    // Streak bonus
    if (currentStreak.value >= STREAK_THRESHOLD) {
      points += STREAK_BONUS
      feedback.value = `✓ Correct! +${points} pts (Streak: ${currentStreak.value})`
    } else if (responseTime < SPEED_BONUS_THRESHOLD) {
      feedback.value = `✓ Correct! +${points} pts (Speed bonus!)`
    } else {
      feedback.value = `✓ Correct! +${points} pts`
    }
    
    score.value += points
    correctAnswers.value++
  } else {
    currentStreak.value = 0
    feedback.value = `✗ Wrong! The answer was ${correctAnswer.value}`
  }
  
  setTimeout(() => {
    generateQuestion()
  }, 1200)
}

function handleKeyPress(event: KeyboardEvent) {
  if (event.key === 'Enter' && gameState.value === 'playing') {
    submitAnswer()
  }
}

onMounted(() => {
  window.addEventListener('keypress', handleKeyPress)
})

onUnmounted(() => {
  if (timerInterval !== null) {
    clearInterval(timerInterval)
  }
  window.removeEventListener('keypress', handleKeyPress)
})
</script>

<template>
  <div class="game-container">
    <h1>Kertolaskupeli</h1>
    
    <div v-if="gameState === 'idle'" class="start-screen">
      <p>Practice your multiplication tables (1-10)!</p>
      <ul>
        <li>Answer as many questions as you can in 2 minutes</li>
        <li>Get {{ BASE_POINTS }} points per correct answer</li>
        <li>Bonus {{ SPEED_BONUS_POINTS }} points for answers under {{ SPEED_BONUS_THRESHOLD }} seconds</li>
        <li>Bonus {{ STREAK_BONUS }} points for {{ STREAK_THRESHOLD }}+ correct streak</li>
      </ul>
      <button @click="startGame" class="btn-primary">Start Game</button>
    </div>

    <div v-if="gameState === 'playing'" class="game-screen">
      <div class="stats">
        <div class="stat">
          <span class="stat-label">Time:</span>
          <span class="stat-value">{{ formattedTime }}</span>
        </div>
        <div class="stat">
          <span class="stat-label">Score:</span>
          <span class="stat-value">{{ score }}</span>
        </div>
        <div class="stat">
          <span class="stat-label">Correct:</span>
          <span class="stat-value">{{ correctAnswers }}</span>
        </div>
        <div class="stat">
          <span class="stat-label">Streak:</span>
          <span class="stat-value" :class="{ 'streak-active': currentStreak >= STREAK_THRESHOLD }">
            {{ currentStreak }}
          </span>
        </div>
      </div>

      <div class="question-area">
        <div class="question">
          <span class="number">{{ num1 }}</span>
          <span class="operator">×</span>
          <span class="number">{{ num2 }}</span>
          <span class="equals">=</span>
        </div>
        
        <input
          v-model="userAnswer"
          type="number"
          class="answer-input"
          placeholder="?"
          autofocus
          :disabled="feedback !== ''"
        />
        
        <button @click="submitAnswer" class="btn-primary" :disabled="!userAnswer || feedback !== ''">
          Submit
        </button>
      </div>

      <div v-if="feedback" class="feedback" :class="{ 'correct': feedback.includes('✓'), 'incorrect': feedback.includes('✗') }">
        {{ feedback }}
      </div>
    </div>

    <div v-if="gameState === 'finished'" class="finish-screen">
      <h2>Game Over!</h2>
      <div class="final-stats">
        <div class="final-stat">
          <span class="final-stat-label">Final Score:</span>
          <span class="final-stat-value">{{ score }}</span>
        </div>
        <div class="final-stat">
          <span class="final-stat-label">Correct Answers:</span>
          <span class="final-stat-value">{{ correctAnswers }}</span>
        </div>
        <div class="final-stat">
          <span class="final-stat-label">Average Points per Answer:</span>
          <span class="final-stat-value">{{ correctAnswers > 0 ? (score / correctAnswers).toFixed(1) : 0 }}</span>
        </div>
      </div>
      <button @click="startGame" class="btn-primary">Play Again</button>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
}

h1 {
  font-size: 2.5rem;
  margin-bottom: 2rem;
  color: #2c3e50;
}

.start-screen, .finish-screen {
  animation: fadeIn 0.5s;
}

.start-screen p {
  font-size: 1.2rem;
  margin-bottom: 1.5rem;
  color: #34495e;
}

.start-screen ul {
  text-align: left;
  display: inline-block;
  margin-bottom: 2rem;
  color: #555;
  line-height: 1.8;
}

.btn-primary {
  background-color: #3498db;
  color: white;
  border: none;
  padding: 1rem 2rem;
  font-size: 1.2rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.btn-primary:hover:not(:disabled) {
  background-color: #2980b9;
}

.btn-primary:disabled {
  background-color: #95a5a6;
  cursor: not-allowed;
}

.stats {
  display: flex;
  justify-content: space-around;
  gap: 1rem;
  margin-bottom: 3rem;
  flex-wrap: wrap;
}

.stat {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.stat-label {
  font-size: 0.9rem;
  color: #7f8c8d;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.stat-value {
  font-size: 1.8rem;
  font-weight: bold;
  color: #2c3e50;
}

.streak-active {
  color: #e74c3c !important;
  animation: pulse 0.5s infinite;
}

.question-area {
  margin: 3rem 0;
}

.question {
  font-size: 3rem;
  font-weight: bold;
  margin-bottom: 2rem;
  color: #2c3e50;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
}

.operator, .equals {
  color: #95a5a6;
}

.answer-input {
  font-size: 2rem;
  padding: 0.8rem;
  width: 150px;
  text-align: center;
  border: 2px solid #bdc3c7;
  border-radius: 8px;
  margin-right: 1rem;
  transition: border-color 0.3s;
}

.answer-input:focus {
  outline: none;
  border-color: #3498db;
}

.answer-input:disabled {
  background-color: #ecf0f1;
}

.feedback {
  margin-top: 1.5rem;
  padding: 1rem;
  border-radius: 8px;
  font-size: 1.2rem;
  font-weight: bold;
  animation: fadeIn 0.3s;
}

.feedback.correct {
  background-color: #d5f4e6;
  color: #27ae60;
}

.feedback.incorrect {
  background-color: #fadbd8;
  color: #e74c3c;
}

.finish-screen h2 {
  font-size: 2rem;
  color: #2c3e50;
  margin-bottom: 2rem;
}

.final-stats {
  margin: 2rem 0;
  padding: 2rem;
  background-color: #f8f9fa;
  border-radius: 8px;
}

.final-stat {
  display: flex;
  justify-content: space-between;
  padding: 1rem;
  border-bottom: 1px solid #dee2e6;
}

.final-stat:last-child {
  border-bottom: none;
}

.final-stat-label {
  font-size: 1.1rem;
  color: #555;
}

.final-stat-value {
  font-size: 1.5rem;
  font-weight: bold;
  color: #2c3e50;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
}
</style>
