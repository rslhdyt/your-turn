<script setup>// import { ref } from 'vue'
import { ref, onMounted, onUnmounted } from 'vue'

const duration = 1000 * 61 * 1

const gameOver = ref(false)
const timeA = ref(duration)
const timeB = ref(duration)

const countdownA = ref('')
const countdownB = ref('')

const audioContext = ref(null)
const soundBuffers = ref({})
const soundSources = ref({})

const soundEffects = {
  tick: '/back-tick.mp3',
  buzz: '/buzz.mp3'
}

const loadTickSound = async () => {
  audioContext.value = new (window.AudioContext || window.webkitAudioContext)()

  for(const [name, path] of Object.entries(soundEffects)) {
    const response = await fetch(path) // Replace with your sound file path
    const arrayBuffer = await response.arrayBuffer()
    soundBuffers.value[name] = await audioContext.value.decodeAudioData(arrayBuffer)
  }
}

const playSound = (name) => {
  if (audioContext.value && soundBuffers.value[name]) {
    stopSound(name)  // Stop any existing tick before playing a new one

    soundSources.value[name] = audioContext.value.createBufferSource()
    soundSources.value[name].buffer = soundBuffers.value[name]
    soundSources.value[name].connect(audioContext.value.destination)
    soundSources.value[name].start()
  }
}

const stopSound = (name) => {
  if (soundSources.value[name]) {
    soundSources.value[name].stop()
    soundSources.value[name] = null
  }
}

const updateCountdown = (time, countdown) => {
  time.value -= 1000
  let diff = time.value
  
  if (diff > 0) {
    const minutes = Math.floor(diff / 60000)
    const seconds = Math.floor((diff % 60000) / 1000)
    countdown.value = `${minutes}:${seconds.toString().padStart(2, '0')}`

    playSound('tick')
  } else {
    stopSound('tick')
    countdown.value = "Time expired\nYou Lose!"
    
    clearInterval(intervalIdA)
    clearInterval(intervalIdB)
    intervalIdA = null
    intervalIdB = null
    
    gameOver.value = true
    playSound('buzz')
  }
}

let intervalIdA = null
let intervalIdB = null

onMounted(async () => {
  await loadTickSound()
  
  startGame()
})

onUnmounted(() => {
  clearInterval(intervalIdA)
  clearInterval(intervalIdB)
})

const startGame = () => {
  timeA.value = duration
  timeB.value = duration

  updateCountdown(timeA, countdownA)
  updateCountdown(timeB, countdownB)

  gameOver.value = false
}

const startCountdownA = () => {
  if (gameOver.value === true) return
  if (intervalIdB !== null) return

  clearInterval(intervalIdA)
  intervalIdA = null

  intervalIdB = setInterval(updateCountdown, 1000, timeB, countdownB)
}

const startCountdownB = () => {
  if (gameOver.value === true) return
  if (intervalIdA !== null) return

  clearInterval(intervalIdB)
  intervalIdB = null

  intervalIdA = setInterval(updateCountdown, 1000, timeA, countdownA)
}
</script>

<template>
  <div class="w-full">
    <div class="h-screen flex flex-col">
      <div class="bg-blue-500 h-1/2" @click="startCountdownA">
        <div class="flex items-center justify-center h-full text-center">
          <span class="text-7xl text-white transform rotate-180 whitespace-pre-line" :class="{ 'text-5xl': (timeA < 0) }">{{ countdownA }}</span>
        </div>
      </div>
      <div class="bg-red-500 h-1/2" @click="startCountdownB">
        <div class="flex items-center justify-center h-full text-center">
          <span class="text-7xl text-white whitespace-pre-line" :class="{ 'text-5xl': (timeB < 0) }">{{ countdownB }}</span>
        </div>
      </div>

      <div class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-white px-14 py-5 text-5xl" 
        @click="startGame"
        v-if="gameOver">
        Start
      </div>
    </div>
  </div>
</template>
