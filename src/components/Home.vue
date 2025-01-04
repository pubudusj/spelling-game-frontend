<script setup>
import { ref } from 'vue'

const words = ref([])
const isLoading = ref(false)
const baseUrl = 'https://ubqh4412yg.execute-api.eu-central-1.amazonaws.com/prod/'
const selectedLanguage = ref('')  // To store selected language
const results = ref([])
const isSubmitted = ref(false)
const timeRemaining = ref(null)
const timer = ref(null)
const TIME_PER_QUESTION = 10

const startTimer = (questionCount) => {
  // Allow seconds per question, max 90 seconds
  const totalSeconds = Math.min(90, questionCount * TIME_PER_QUESTION)
  timeRemaining.value = totalSeconds
  
  timer.value = setInterval(() => {
    if (timeRemaining.value > 0) {
      timeRemaining.value--
    } else {
      if (!isSubmitted.value) {
        handleSubmit() // Auto-submit if not already submitted
      }
      clearInterval(timer.value)
    }
  }, 1000)
}
const fetchWords = async (language) => {
  isLoading.value = true
  try {
    const headers = new Headers({
      'Content-Type': 'application/json',
    })
    const response = await fetch(`${baseUrl}questions`, {
      method: 'POST',
      headers: headers,
      body: JSON.stringify({ language })
    })
    const data = await response.json()
    words.value = data['questions']
    startTimer(data['questions'].length) // Start timer when questions are loaded
  } catch (error) {
    console.error('Error fetching words:', error)
  } finally {
    isLoading.value = false
  }
}

const handleLanguageSelect = (language) => {
  selectedLanguage.value = language
  // Reset form state
  isSubmitted.value = false
  results.value = []
  words.value = []
  timeRemaining.value = null
  if (timer.value) {
    clearInterval(timer.value)
    timer.value = null
  }
  fetchWords(language)
}

const handleAudioError = (event) => {
  console.error('Error loading audio:', event)
}

const handleInput = (event, index, wordId) => {
  const inputs = event.target.parentElement.getElementsByTagName('input')
  if (event.target.value.length >= 1) {
    // Move to next input if available
    if (index < inputs.length) {
      inputs[index]?.focus()
    }
  } else if (event.key === 'Backspace' && index > 1) {
    // Move to previous input on backspace
    inputs[index - 2]?.focus()
  }
}

const enforceMaxLength = (input) => {
  if (input.value.length > 1) {
    input.value = input.value.slice(0, 1)
  }
}

// Function to collect answers from inputs
const collectAnswers = () => {
  return words.value.map(word => {
    const inputContainer = document.querySelector(`[data-word-id="${word.id}"]`)
    const inputs = inputContainer?.getElementsByTagName('input')
    const letters = inputs ? Array.from(inputs).map(input => input.value).join('') : ''
    
    return {
      id: word.id,
      word: letters
    }
  })
}

// Function to handle form submission
const handleSubmit = async () => {
  const answers = collectAnswers()
  const payload = {
    language: selectedLanguage.value,
    answers: answers
  }
  isSubmitted.value = true
  clearInterval(timer.value) // Stop the timer when form is submitted
  try {
    const response = await fetch(`${baseUrl}answers`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(payload)
    })
    const data = await response.json()
    results.value = data
  } catch (error) {
    console.error('Error submitting answers:', error)
  }
}
</script>

<template>
  <div class="home">
    <div class="fixed-header">
      <h1 class="main-title">Check Your Spelling</h1>
      <h2 class="subtitle">Select a language to start a new game</h2>
    </div>
    <div style="height: 130px"></div>
    
    <div class="language-buttons">
      <button 
        class="language-btn"
        @click="handleLanguageSelect('en-US')"
        :disabled="isLoading"
      >
        English
      </button>
      <button 
        class="language-btn"
        @click="handleLanguageSelect('nl-NL')"
        :disabled="isLoading"
      >
        Dutch
      </button>
    </div>

    <!-- Display loading state -->
    <div v-if="isLoading" class="loading">
      Loading...
    </div>
    
    <div class="questions-container">
      <!-- Display timer -->
      <div v-if="timeRemaining !== null" class="timer" :class="{ 'warning': timeRemaining < 30 }">
        Time remaining: {{ Math.floor(timeRemaining / 60) }}:{{ (timeRemaining % 60).toString().padStart(2, '0') }}
      </div>

      <!-- Display words after API call -->
      <div v-if="words.length > 0" class="words-list">
        <div v-for="word in words" :key="word.id" class="word-item">
          <div class="audio-container">
            <audio
              :src="word.url"
              controls
              class="audio-player"
              @error="handleAudioError"
            >
              Your browser does not support the audio element.
            </audio>
          </div>
          <p>{{ word.description }}</p>
            <div class="input-container" :data-word-id="word.id" :class="{
              'correct': isSubmitted && results.find(r => r.id === word.id)?.correct,
              'incorrect': isSubmitted && !results.find(r => r.id === word.id)?.correct 
            }">
              <div class="input-container" :data-word-id="word.id">
              <input
                v-for="index in word.charcount"
                :key="index"
                type="text"
                maxlength="1"
                class="letter-input"
                @keyup="handleInput($event, index, word.id)"
                @input="enforceMaxLength($event.target)"
                :disabled="isSubmitted"
              >
              <div v-if="isSubmitted" class="result-text">
                {{ results.find(r => r.id === word.id)?.correct ? 'Correct!' : 'Incorrect' }}
                <span class="original-word" v-if="!results.find(r => r.id === word.id)?.correct">
                  ({{ results.find(r => r.id === word.id)?.original_word }})
                </span>
              </div>
            </div>
          </div>
        </div>

        <!-- Single submit button outside the word loop -->
        <div class="submit-container">
          <button 
              class="submit-btn"
              @click="handleSubmit"
              :disabled="isLoading || isSubmitted"
            >
              {{ isSubmitted ? 'Submitted' : 'Submit Answers' }}
            </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.home {
  text-align: center;
  padding: 2rem;
  padding-top: 0;
}

.fixed-header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  background-color: #1a1a1a;
  padding: 1rem;
  z-index: 100;
}

.main-title {
  font-size: 3rem;
  margin-bottom: 1rem;
  color: #6189b1;
}

.subtitle {
  font-size: 1.5rem;
  margin-bottom: 2rem;
  color: #ffffff;
}

.language-buttons {
  display: flex;
  gap: 1rem;
  justify-content: center;
  position: fixed;
  top: 150px;
  left: 0;
  right: 0;
  z-index: 100;
  background-color: #1a1a1a;
  padding: 1rem;
}

.language-btn {
  padding: 1rem 2rem;
  font-size: 1.2rem;
  border: none;
  border-radius: 8px;
  background-color: #42b883;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
}

.language-btn:hover {
  background-color: #3aa876;
}

.language-btn:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.loading {
  margin-top: 1rem;
  color: #666;
}

.words-list {
  top: 550px;
  max-width: 600px;
  margin: 0 auto;
}

.word-item {
  background-color: #fff;
  padding: 2rem;
  margin-bottom: 1.5rem;
  border-radius: 12px;
  max-width: 800px;
  margin-left: auto;
  margin-right: auto;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.word-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
}

.audio-container {
  margin: 1rem 0;
}

.audio-player {
  width: 100%;
  max-width: 300px;
  height: 40px;
  border-radius: 4px;
}

.input-container {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  align-items: center;
  margin: 1rem 0;
}

.letter-input {
  width: 40px;
  height: 40px;
  text-align: center;
  font-size: 1.2rem;
  border: 2px solid #5884ec;
  border-radius: 4px;
  outline: none;
}

.letter-input:focus {
  border-color: #3aa876;
  box-shadow: 0 0 5px rgba(66, 184, 131, 0.5);
}
.submit-container {
    margin-top: 2rem;
    display: flex;
    justify-content: center;
}

.submit-btn {
    padding: 1rem 2rem;
    font-size: 1.2rem;
    border: none;
    border-radius: 8px;
    background-color: #42b883;
    color: white;
    cursor: pointer;
    transition: background-color 0.3s;
}

.submit-btn:hover {
    background-color: #3aa876;
}

.submit-btn:disabled {
    background-color: #ccc;
    cursor: not-allowed;
}

.input-container.correct .letter-input {
  background-color: rgba(76, 175, 80, 0.1);
  border-color: #4CAF50;
}

.input-container.incorrect .letter-input {
  background-color: rgba(244, 67, 54, 0.1);
  border-color: #F44336;
}

.letter-input:disabled {
  background-color: #f5f5f5;
  cursor: not-allowed;
}

.result-text {
  margin-left: 1rem;
  font-weight: bold;
}

.input-container.correct .result-text {
  color: #4CAF50;
}

.input-container.incorrect .result-text {
  color: #F44336;
}

.original-word {
  font-style: italic;
  margin-left: 0.5rem;
  font-weight: normal;
}

.timer {
  font-size: 1.5rem;
  margin: 1rem 0;
  padding: 0.5rem;
  background-color: #42b883;
  color: white;
  border-radius: 4px;
  display: inline-block;
}

.timer.warning {
  background-color: #ff4444;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0% { opacity: 1; }
  50% { opacity: 0.7; }
  100% { opacity: 1; }
}

.questions-container {
    position: absolute;
    top: 250px;
    left: 0;
    right: 0;
}
</style>
