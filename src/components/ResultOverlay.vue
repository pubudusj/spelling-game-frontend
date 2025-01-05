<script setup>
import { computed } from 'vue'
import ConfettiEffect from './ConfettiEffect.vue'

const props = defineProps({
  totalQuestions: {
    type: Number,
    required: true
  },
  correctCount: {
    type: Number,
    required: true
  },
  isVisible: {
    type: Boolean,
    required: true
  }
})

const isPerfectScore = computed(() => {
  const result = props.correctCount === props.totalQuestions
  // console.log('isPerfectScore computed:', result, 'correctCount:', props.correctCount, 'total:', props.totalQuestions) // Debug log
  return result
})

const emit = defineEmits(['close'])
</script>

<template>
  <div v-if="isVisible" class="overlay">
    <ConfettiEffect v-if="isPerfectScore" :active="true" />
    <div class="overlay-content">
      <h2>Quiz Results</h2>
      <p class="result-summary">
        You got <span class="correct-count">{{ correctCount }}</span> out of 
        <span class="total-count">{{ totalQuestions }}</span> questions correct!
      </p>
      <button class="close-btn" @click="$emit('close')">View Results</button>
    </div>
  </div>
</template>

<style scoped>
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.85);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 99000;
}

.overlay-content {
  background-color: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  text-align: center;
  max-width: 400px;
  width: 90%;
}

h2 {
  color: #42b883;
  margin-bottom: 1rem;
}

.result-summary {
  font-size: 1.2rem;
  margin-bottom: 1.5rem;
}

.correct-count {
  color: #42b883;
  font-weight: bold;
  font-size: 1.5rem;
}

.total-count {
  font-weight: bold;
}

.close-btn {
  padding: 0.5rem 1.5rem;
  font-size: 1rem;
  border: none;
  border-radius: 4px;
  background-color: #42b883;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
}

.close-btn:hover {
  background-color: #3aa876;
}
</style>