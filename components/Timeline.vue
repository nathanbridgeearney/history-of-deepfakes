<template>
  <div class="h-screen flex flex-col bg-gradient-to-br from-sky-100 to-amber-100 dark:from-sky-950 dark:to-amber-950" @wheel.passive="handleWheel" tabindex="0" @keydown="handleKeyDown" ref="container">
    <div class="flex-1 overflow-hidden bg-sky-100 dark:bg-sky-950">
      <transition name="fade" mode="out-in" @before-enter="lockNavigation" @after-leave="unlockNavigation">
        <Applications :key="currentIndex" :current-item="currentApplication" />
      </transition>
    </div>
    <div
      ref="scrollContainer"
      class="h-12 bg-gradient-to-r from-sky-200 to-amber-200 dark:from-sky-900 dark:to-amber-900 relative flex items-center overflow-hidden"
    >
      <button 
        @click="navigatePrev" 
        class="absolute left-0 top-0 bottom-0 w-6 bg-sky-300 hover:bg-sky-400 dark:bg-sky-800 dark:hover:bg-sky-700 flex items-center justify-center z-10 text-sky-900 dark:text-white"
        :disabled="currentIndex <= 0 || isAnimating"
      >
        &lt;
      </button>
      <div class="absolute left-6 right-6 top-0 bottom-0 flex">
        <div 
          v-for="(item, index) in limitedTimelineData" 
          :key="index"
          class="flex-1 h-full border-r border-sky-300 dark:border-sky-800 last:border-r-0 relative cursor-pointer overflow-hidden"
          @click="jumpToStep(index)"
        >
          <div 
            class="absolute top-0 left-0 h-full w-full transition-transform duration-300 ease-in-out bg-gradient-to-r from-sky-400 to-amber-400 dark:from-sky-500 dark:to-amber-500"
            :style="getStepStyle(index)"
          ></div>
          <div class="absolute inset-0 flex items-center justify-center text-xs font-semibold text-sky-900 dark:text-white z-10">
            {{ item.stepLabel }}
          </div>
        </div>
      </div>
      <div 
        class="absolute top-0 left-0 h-full bg-gradient-to-r from-sky-400 to-amber-400 dark:from-sky-500 dark:to-amber-500 transition-all duration-300 ease-in-out"
        :style="{ width: `${(currentIndex + 1) * (100 / limitedTimelineData.length)}%` }"
      ></div>
      <button 
        @click="navigateNext" 
        class="absolute right-0 top-0 bottom-0 w-6 bg-amber-300 hover:bg-amber-400 dark:bg-amber-800 dark:hover:bg-amber-700 flex items-center justify-center z-10 text-amber-900 dark:text-white"
        :disabled="currentIndex >= limitedTimelineData.length - 1 || isAnimating"
      >
        &gt;
      </button>
    </div>
    <div class="flex-1 overflow-hidden bg-amber-100 dark:bg-amber-950">
      <transition name="fade" mode="out-in" @before-enter="lockNavigation" @after-leave="unlockNavigation">
        <Implications :key="currentIndex" :current-item="currentImplication" />
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useAsyncData } from '#app'
import Applications from './Applications.vue'
import Implications from './Implications.vue'

const container = ref(null)
const scrollContainer = ref(null)
const currentIndex = ref(0)
const isAnimating = ref(false)
const wheelAccumulator = ref(0)
const WHEEL_THRESHOLD = 100 // Scroll Smoothness
const DEBOUNCE_DELAY = 50 // Scroll Delay

const { data: timelineData } = await useAsyncData('timeline', () => queryContent('/').find())

const limitedTimelineData = computed(() => {
  return timelineData.value.map(item => ({
    stepLabel: item.stepLabel,
    application: {
      title: item.applicationTitle,
      description: item.applicationDescription
    },
    implication: {
      title: item.implicationTitle,
      description: item.implicationDescription
    }
  })).slice(0, 6)
})

const totalSets = computed(() => limitedTimelineData.value.length)

const currentApplication = computed(() => limitedTimelineData.value[currentIndex.value].application)
const currentImplication = computed(() => limitedTimelineData.value[currentIndex.value].implication)

const getStepStyle = (index) => {
  if (index < currentIndex.value) {
    return { transform: 'scaleX(1)' }
  } else if (index === currentIndex.value) {
    return { transform: 'scaleX(1)' }
  }
  return { transform: 'scaleX(0)' }
}

let debounceTimer = null

const handleWheel = (e) => {
  e.preventDefault()
  if (isAnimating.value) return

  wheelAccumulator.value += Math.abs(e.deltaX) > Math.abs(e.deltaY) ? e.deltaX : e.deltaY

  clearTimeout(debounceTimer)
  debounceTimer = setTimeout(() => {
    if (Math.abs(wheelAccumulator.value) >= WHEEL_THRESHOLD) {
      const direction = wheelAccumulator.value > 0 ? 1 : -1
      setCurrentIndex(currentIndex.value + direction)
      wheelAccumulator.value = 0
    }
  }, DEBOUNCE_DELAY)
}

const handleKeyDown = (e) => {
  if (isAnimating.value) return
  if (e.key === 'ArrowLeft') {
    navigatePrev()
  } else if (e.key === 'ArrowRight') {
    navigateNext()
  }
}

const setCurrentIndex = (newIndex) => {
  if (isAnimating.value) return
  currentIndex.value = Math.max(0, Math.min(Math.round(newIndex), totalSets.value - 1))
}

const navigatePrev = () => {
  if (isAnimating.value || currentIndex.value <= 0) return
  setCurrentIndex(currentIndex.value - 1)
}

const navigateNext = () => {
  if (isAnimating.value || currentIndex.value >= totalSets.value - 1) return
  setCurrentIndex(currentIndex.value + 1)
}

const jumpToStep = (index) => {
  if (isAnimating.value) return
  setCurrentIndex(index)
}

const lockNavigation = () => {
  isAnimating.value = true
}

const unlockNavigation = () => {
  isAnimating.value = false
}

onMounted(() => {
  window.addEventListener('resize', handleResize)
  container.value.focus() 
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  clearTimeout(debounceTimer)
})

const handleResize = () => {
  // TODO
}
</script>

<style scoped>
.h-screen {
  height: 100vh;
  outline: none; 
  user-select: none;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease; 
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

button:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}
</style>