<script setup>
import { ref } from 'vue'

const words = ref([])
const isLoading = ref(false)
const baseUrl = 'https://ubqh4412yg.execute-api.eu-central-1.amazonaws.com/prod/'
const selectedLanguage = ref('')  // To store selected language
const results = ref([])
const isSubmitted = ref(false)
console.log(results)
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
  } catch (error) {
    console.error('Error fetching words:', error)
  } finally {
    isLoading.value = false
  }
}

const handleLanguageSelect = (language) => {
  selectedLanguage.value = language
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
    <h1 class="main-title">Spelling Game</h1>
    <h2 class="subtitle">Start a new game</h2>
    
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

    <!-- Display words after API call -->
    <div v-if="words.length > 0" class="words-list">
      <div v-for="word in words" :key="word.id" class="word-item">
        <p>{{ word.description }}</p>
        <p>Characters: {{ word.charcount }}</p>

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
            <span v-if="isSubmitted" class="result-text">
              {{ results.find(r => r.id === word.id)?.correct ? 'Correct!' : 'Incorrect' }}
              <span class="original-word" v-if="!results.find(r => r.id === word.id)?.correct">
                ({{ results.find(r => r.id === word.id)?.original_word }})
              </span>
            </span>
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
</template>

<style scoped>
.home {
  text-align: center;
  padding: 2rem;
}

.main-title {
  font-size: 3rem;
  margin-bottom: 1rem;
  color: #2c3e50;
}

.subtitle {
  font-size: 1.5rem;
  margin-bottom: 2rem;
  color: #666;
}

.language-buttons {
  display: flex;
  gap: 1rem;
  justify-content: center;
  margin-bottom: 2rem;
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
  max-width: 600px;
  margin: 0 auto;
}

.word-item {
  background-color: #525252;
  padding: 1rem;
  margin-bottom: 1rem;
  border-radius: 8px;
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
  gap: 0.5rem;
  justify-content: center;
  margin: 1rem 0;
}

.letter-input {
  width: 40px;
  height: 40px;
  text-align: center;
  font-size: 1.2rem;
  border: 2px solid #42b883;
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

</style>
