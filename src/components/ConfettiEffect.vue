<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'

const props = defineProps({
  active: {
    type: Boolean,
    required: true
  }
})

let confettiContainer = null
let animationFrame = null

const colors = ['#ffd700', '#42b883', '#ff69b4', '#00bfff', '#ff4500', '#9b59b6', '#2ecc71', '#f1c40f', '#e74c3c', '#3498db']

const setupContainer = () => {
  if (!confettiContainer) {
    confettiContainer = document.createElement('div')
    confettiContainer.className = 'confetti-container'
    document.body.appendChild(confettiContainer)
  }
}

watch(() => props.active, (newValue) => {
  if (newValue) {
    setupContainer()
    requestAnimationFrame(animate)
  } else {
    cleanup()
  }
}, { immediate: true })

function createParticle() {
  const particle = document.createElement('div')
  particle.className = 'confetti'
  particle.style.backgroundColor = colors[Math.floor(Math.random() * colors.length)]
  particle.style.boxShadow = '0 0 6px rgba(0,0,0,0.15)'
  particle.style.left = Math.random() * window.innerWidth + 'px'
  particle.style.animationDuration = (Math.random() * 4 + 3) + 's'
  particle.style.opacity = Math.random() + 0.5
  // Randomly choose between circle and square
  if (Math.random() > 0.5) {
    particle.style.borderRadius = '50%'
  }
  // Randomly adjust size
  const size = Math.random() * 12 + 8 // 8-20px
  particle.style.width = size + 'px'
  particle.style.height = size + 'px'
  return particle
}

function animate() {
  if (!props.active || !confettiContainer) {
    console.log('Animation stopped:', { active: props.active, hasContainer: !!confettiContainer });
    return;
  }

  // Create large initial burst of particles
  const particleCount = animationFrame === null ? 250 : 25
  
  // Create multiple particles at once
  for (let i = 0; i < particleCount; i++) {
    const particle = createParticle()
    confettiContainer.appendChild(particle)

    // Remove particle after animation
    setTimeout(() => {
      if (particle.parentNode === confettiContainer) {
        confettiContainer.removeChild(particle)
      }
    }, 6000)
  }

  // Schedule next frame
  animationFrame = requestAnimationFrame(animate)
}

const cleanup = () => {
  if (confettiContainer) {
    document.body.removeChild(confettiContainer)
    confettiContainer = null
  }
  if (animationFrame) {
    cancelAnimationFrame(animationFrame)
    animationFrame = null
  }
}

onMounted(() => {
  if (props.active) {
    setupContainer()
    requestAnimationFrame(animate)
  }
})

onUnmounted(() => {
  cleanup()
})
</script>

<template>
  <!-- Component is renderless, container managed in JS -->
</template>

<style>
.confetti-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 100000;
  overflow: hidden;
  pointer-events: none;
  background: transparent;
  will-change: transform;
}

.confetti {
  position: absolute;
  width: 10px;
  height: 10px;
  animation: fall 5s cubic-bezier(0.445, 0.05, 0.55, 0.95) forwards;
  transform-origin: center center;
}

@keyframes fall {
  0% {
    transform: translateY(-100vh) rotate(0deg) scale(0);
    opacity: 0;
  }
  10% {
    transform: translateY(-50vh) rotate(45deg) scale(1);
    opacity: 1;
  }
  45% {
    transform: translateY(0) rotate(180deg) scale(1);
    opacity: 1;
  }
  100% {
    transform: translateY(100vh) rotate(720deg) scale(0.5);
    opacity: 0;
  }
}
</style>