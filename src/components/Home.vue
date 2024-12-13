<script setup>
import { ref } from 'vue'

const words = ref([])
const isLoading = ref(false)
const baseUrl = 'https://ubqh4412yg.execute-api.eu-central-1.amazonaws.com/prod/'

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
  fetchWords(language)
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
        <p>Characters: {{ word.character_count }}</p>
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
  background-color: #f5f5f5;
  padding: 1rem;
  margin-bottom: 1rem;
  border-radius: 8px;
}
</style>
