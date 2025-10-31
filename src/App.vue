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
const answerInputRef = ref<HTMLInputElement | null>(null)

const num1 = ref(0)
const num2 = ref(0)
const feedback = ref('')

let timerInterval: number | null = null

// Horse sound audio elements
const correctSound = new Audio('data:audio/wav;base64,UklGRnoGAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YQoGAACAgoSGiIqMjpCSk5SVlpeYmZqbnJ2en6ChoqOkpaanqKmqq6ytrq+wsbKztLW2t7i5uru8vb6/wMHCw8TFxsfIycrLzM3Oz9DR0tPU1dbX2Nna29zd3t/g4eLj5OXm5+jp6uvs7e7v8PHy8/T19vf4+fr7/P3+//7+/f39/Pz7+/r6+fn4+Pf39vb19PTz8/Ly8fHw8O/v7u7t7ezs6+vq6uno6Ofn5uXl5OTj4uLh4ODf397e3d3c3Nvb2tnZ2NjX19bV1dTU09PS0tHRz8/Ozs3NzMvLysrJycjHx8bGxcTExMPCwsHBwL+/vr69vby8u7q6ubm4t7e2trW0tLOzsrGxsK+vrq6trKyrq6qpqainp6ampaSkpKOioaGgoJ+enp2dnJubmpqZmJiXl5aVlZSUk5KSkZCQj4+Ojo2MjIuKioiIh4eGhYWEhIOCgoGAgH9/fn59fXx7e3p6eXl4d3d2dXV0dHNycXBwb29ubW1sbGtqamlpaGdnZmZlZGRjY2JhYWBgX15eXV1cW1tbWllaWVhYV1dWVVVUVFNSUlFRUFBPT05NTU1MTEtKSklJSEhHR0ZGRUVERENDQkJBQEBAQD8/Pj49PT08PDo6OTk4ODc3NjY1NTQ0MzMyMjExMDAvLy4uLS0sLCsrKioqKSkoKCcnJiYlJSQkIyMiIiEhICAfHx4eHR0cHBsbGhoZGBgXFxYWFRUUFBMTEhIRERAQDw4ODQ0MDAsLCgoJCQgIBwYGBQUEBAMDAgIBAQAAAAAAAQICAwMEBAUFBgYHCAgJCQoKCwwMDQ0ODg8QEBESEhMTFBQVFRYWFxgYGRoaGxscHB0dHh8fICEhIiIjJCQlJSYnJygpKSoqKyssLS0uLi8wMDExMjMzNDU1NjY3Nzg5OTo7Ozw9PT4/P0BBQUJCREREBQVGRkdISEpKS0tMTU1OT1BQUVJSVFRVVVdXWFlaWltcXF1eXmBgYWJiY2RkZWZnZ2hpaWpra2xtbW5vb3BxcnJzc3V1dnd4eHl5e3t8fX1+f4CAgYKChISFhoeHiImJioyMjY6Oj5CRkZKTk5SUlpaXmJiZmpqbnJydnp6foKChoqKkpKWmp6eoqamqq6ytre7u7u/w8PHx8vLz8/T09fX29vf4+Pn5+vr7+/z8/f3+/v//') 
const incorrectSound = new Audio('data:audio/wav;base64,UklGRiQEAABXQVZFZm10IBAAAAABAAEARKwAAIhYAQACABAAZGF0YQAEAAB/f39/f39/f39/gICAgICAgICBgYGBgYKCgoKCg4ODg4OEhISEhYWFhYaGhoaHh4eHiIiIiImJiYmKioqKi4uLi4yMjIyNjY2Njo6Ojo+Pj4+QkJCQkZGRkZKSkpKTk5OTlJSUlJWVlZWWlpaWl5eXl5iYmJiZmZmZmpqampubm5ucnJycnZ2dnZ6enp6fn5+foKCgoKGhoaGioqKio6Ojo6SkpKSlpaWlpqamp6enp6ioqKipqamqqqqqq6urq6ysrKytra2trq6urq+vr7CwsLCxsbGxsrKysrOzs7S0tLS1tbW1tra2t7e3t7i4uLm5ubm6urq7u7u7vLy8vL29vb6+vr6/v7/AwMDAwcHBwsLCwsPDw8TExMTFxcXGxsbGx8fHyMjIyMnJycrKysrLy8vMzMzMzc3Nzs7Ozs/Pz9DQ0NDR0dHR0tLS09PT1NTU1NXV1dbW1tfX19fY2NjZ2dna2trbW1tb3Nzc3d3d3t7e3t/f3+Dg4OHh4eHi4uLj4+Pk5OTk5eXl5ubm5+fn6Ojo6Onp6erq6uvr6+zs7Ozt7e3u7u7v7+/v8PDw8fHx8vLy8/Pz9PT09PX19fb29vf39/j4+Pn5+fr6+vv7+/z8/P39/f7+/v//////////////////////////////////////////////////////////////////////////////////////////////f39+fn59fX18fHx7e3t6enp5eXl4eHh3d3d2dnZ1dXV0dHRzc3NycnJxcXBwcG9vb25ubm1tbWxsbGtra2pqamlpaWhoaGdnZ2ZmZmVlZWRkZGNjY2JiYmFhYGBgX19fXl5eXV1dXFxcW1tbWlpaWVlZWFhYV1dXVlZWVVVVVFRUU1NTUlJSUVFRUFBQT09PTk5OTU1NTExMS0tLSkpKSUlJSEhIR0dHRkZGRUVFRERDQ0NCQkJBQUFAPz8/Pj4+PT09PDw8Ozs7Ojo6OTk5ODg4Nzc3NjY2NTU1NDQ0MzMzMjIyMTExMDAwLy8vLi4uLS0tLCwsKysrKioqKSkpKCgoJycnJiYmJSUlJCQkIyMjIiIiISEhICAgHx8fHh4eHR0dHBwcGxsbGhoZGRkYGBgXFxcWFhYVFRUUFBQTExMSEhIREREQEBAPDw8ODg4NDQ0MDAwLCwsKCgoJCQkICAcHBwYGBgUFBQQEAwMDAgICAQEBAAAAAAAA')

function playCorrectSound() {
  correctSound.currentTime = 0
  correctSound.play().catch(() => {}) // Ignore errors if autoplay is blocked
}

function playIncorrectSound() {
  incorrectSound.currentTime = 0
  incorrectSound.play().catch(() => {}) // Ignore errors if autoplay is blocked
}

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
  
  // Focus the input on next tick
  setTimeout(() => {
    answerInputRef.value?.focus()
  }, 0)
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
    
    // Play correct sound
    playCorrectSound()
    
    // Streak bonus
    if (currentStreak.value >= STREAK_THRESHOLD) {
      points += STREAK_BONUS
      feedback.value = `🐴 Oikein! +${points} p (Putki: ${currentStreak.value}) 🏇`
    } else if (responseTime < SPEED_BONUS_THRESHOLD) {
      feedback.value = `🐴 Oikein! +${points} p (Nopea kuin hevonen!) 🏇`
    } else {
      feedback.value = `🐴 Oikein! +${points} p`
    }
    
    score.value += points
    correctAnswers.value++
  } else {
    currentStreak.value = 0
    playIncorrectSound()
    feedback.value = `🐴 Väärin! Oikea vastaus oli ${correctAnswer.value}`
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
    <h1>🐴 Hevosen Kertolaskupeli 🐴</h1>
    
    <div v-if="gameState === 'idle'" class="start-screen">
      <div class="horse-emoji">🐎</div>
      <p>Laukka kertolaskujen maailmaan!</p>
      <ul>
        <li>🏇 Vastaa mahdollisimman moneen kysymykseen 2 minuutissa</li>
        <li>🥕 {{ BASE_POINTS }} pistettä jokaisesta oikeasta vastauksesta</li>
        <li>⚡ {{ SPEED_BONUS_POINTS }} bonuspistettä vastauksista alle {{ SPEED_BONUS_THRESHOLD }} sekunnissa</li>
        <li>🏆 {{ STREAK_BONUS }} bonuspistettä {{ STREAK_THRESHOLD }}+ oikean vastauksen putkilta</li>
      </ul>
      <button @click="startGame" class="btn-primary">🐴 Aloita laukka!</button>
    </div>

    <div v-if="gameState === 'playing'" class="game-screen">
      <div class="stats">
        <div class="stat">
          <span class="stat-label">⏱️ Aika:</span>
          <span class="stat-value">{{ formattedTime }}</span>
        </div>
        <div class="stat">
          <span class="stat-label">🥕 Pisteet:</span>
          <span class="stat-value">{{ score }}</span>
        </div>
        <div class="stat">
          <span class="stat-label">✅ Oikein:</span>
          <span class="stat-value">{{ correctAnswers }}</span>
        </div>
        <div class="stat">
          <span class="stat-label">🏆 Putki:</span>
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
          ref="answerInputRef"
          type="number"
          class="answer-input"
          placeholder="?"
          :disabled="feedback !== ''"
        />
        
        <button @click="submitAnswer" class="btn-primary" :disabled="!userAnswer || feedback !== ''">
          Vastaa
        </button>
      </div>

      <div v-if="feedback" class="feedback" :class="{ 'correct': feedback.includes('🐴') && feedback.includes('Oikein'), 'incorrect': feedback.includes('🐴') && feedback.includes('Väärin') }">
        {{ feedback }}
      </div>
    </div>

    <div v-if="gameState === 'finished'" class="finish-screen">
      <div class="horse-emoji">🏇</div>
      <h2>Maali saavutettu! 🎉</h2>
      <div class="final-stats">
        <div class="final-stat">
          <span class="final-stat-label">🥕 Lopulliset pisteet:</span>
          <span class="final-stat-value">{{ score }}</span>
        </div>
        <div class="final-stat">
          <span class="final-stat-label">✅ Oikeat vastaukset:</span>
          <span class="final-stat-value">{{ correctAnswers }}</span>
        </div>
        <div class="final-stat">
          <span class="final-stat-label">📊 Keskimäärin pisteitä per vastaus:</span>
          <span class="final-stat-value">{{ correctAnswers > 0 ? (score / correctAnswers).toFixed(1) : 0 }}</span>
        </div>
      </div>
      <button @click="startGame" class="btn-primary">🐴 Laukka uudelleen!</button>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
  background: linear-gradient(135deg, #f5e6d3 0%, #e8d4b8 100%);
  border-radius: 16px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
  border: 4px solid #8b6914;
}

h1 {
  font-size: 2.5rem;
  margin-bottom: 2rem;
  color: #8b4513;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

.horse-emoji {
  font-size: 4rem;
  margin-bottom: 1rem;
  animation: bounce 1s infinite;
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

.start-screen, .finish-screen {
  animation: fadeIn 0.5s;
}

.start-screen p {
  font-size: 1.2rem;
  margin-bottom: 1.5rem;
  color: #654321;
  font-weight: 600;
}

.start-screen ul {
  text-align: left;
  display: inline-block;
  margin-bottom: 2rem;
  color: #654321;
  line-height: 1.8;
}

.btn-primary {
  background: linear-gradient(135deg, #d2691e 0%, #a0522d 100%);
  color: white;
  border: none;
  padding: 1rem 2rem;
  font-size: 1.2rem;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
  font-weight: 600;
}

.btn-primary:hover:not(:disabled) {
  background: linear-gradient(135deg, #a0522d 0%, #8b4513 100%);
  transform: translateY(-2px);
  box-shadow: 0 6px 8px rgba(0, 0, 0, 0.3);
}

.btn-primary:disabled {
  background: #95a5a6;
  cursor: not-allowed;
  transform: none;
}

.stats {
  display: flex;
  justify-content: space-around;
  gap: 1rem;
  margin-bottom: 3rem;
  flex-wrap: wrap;
  background-color: rgba(255, 255, 255, 0.7);
  padding: 1.5rem;
  border-radius: 12px;
  border: 2px solid #d2691e;
}

.stat {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.stat-label {
  font-size: 0.9rem;
  color: #654321;
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: 600;
}

.stat-value {
  font-size: 1.8rem;
  font-weight: bold;
  color: #8b4513;
}

.streak-active {
  color: #ff6b35 !important;
  animation: pulse 0.5s infinite;
  text-shadow: 0 0 10px rgba(255, 107, 53, 0.5);
}

.question-area {
  margin: 3rem 0;
}

.question {
  font-size: 3rem;
  font-weight: bold;
  margin-bottom: 2rem;
  color: #8b4513;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  background-color: rgba(255, 255, 255, 0.7);
  padding: 1.5rem;
  border-radius: 12px;
  border: 2px solid #d2691e;
}

.operator, .equals {
  color: #d2691e;
}

.answer-input {
  font-size: 2rem;
  padding: 0.8rem;
  width: 150px;
  text-align: center;
  border: 3px solid #d2691e;
  border-radius: 8px;
  margin-right: 1rem;
  transition: border-color 0.3s;
  background-color: rgba(255, 255, 255, 0.9);
}

.answer-input:focus {
  outline: none;
  border-color: #8b4513;
  box-shadow: 0 0 10px rgba(210, 105, 30, 0.3);
}

.answer-input:disabled {
  background-color: #ecf0f1;
}

.feedback {
  margin-top: 1.5rem;
  padding: 1rem;
  border-radius: 8px;
  font-size: 1.3rem;
  font-weight: bold;
  animation: fadeIn 0.3s;
  border: 3px solid;
}

.feedback.correct {
  background: linear-gradient(135deg, #90EE90 0%, #98FB98 100%);
  color: #1e8449;
  font-weight: 600;
  border-color: #228B22;
  box-shadow: 0 4px 6px rgba(34, 139, 34, 0.3);
}

.feedback.incorrect {
  background: linear-gradient(135deg, #FFB6C1 0%, #FFC0CB 100%);
  color: #c0392b;
  font-weight: 600;
  border-color: #8B0000;
  box-shadow: 0 4px 6px rgba(139, 0, 0, 0.3);
}

.finish-screen h2 {
  font-size: 2rem;
  color: #8b4513;
  margin-bottom: 2rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

.final-stats {
  margin: 2rem 0;
  padding: 2rem;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.9) 0%, rgba(255, 248, 240, 0.9) 100%);
  border-radius: 8px;
  border: 3px solid #d2691e;
}

.final-stat {
  display: flex;
  justify-content: space-between;
  padding: 1rem;
  border-bottom: 2px solid #d2691e;
}

.final-stat:last-child {
  border-bottom: none;
}

.final-stat-label {
  font-size: 1.1rem;
  color: #654321;
  font-weight: 500;
}

.final-stat-value {
  font-size: 1.5rem;
  font-weight: bold;
  color: #8b4513;
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
