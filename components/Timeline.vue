<template>
  <div class="h-screen flex flex-col" @wheel.passive="handleWheel" tabindex="0" @keydown="handleKeyDown" ref="container">
    <div class="flex-1 overflow-hidden">
      <transition name="fade" mode="out-in" @before-enter="lockNavigation" @after-leave="unlockNavigation">
        <Applications :key="currentIndex" :current-item="currentApplication" />
      </transition>
    </div>
    <div
      ref="scrollContainer"
      class="h-12 bg-gray-200 relative flex items-center"
    >
      <button 
        @click="navigatePrev" 
        class="absolute left-0 top-0 bottom-0 w-6 bg-gray-300 hover:bg-gray-400 flex items-center justify-center z-10"
        :disabled="currentIndex <= 0 || isAnimating"
      >
        &lt;
      </button>
      <div class="absolute left-6 right-6 top-0 bottom-0 flex">
        <div 
          v-for="(item, index) in timelineData" 
          :key="index"
          class="flex-1 h-full border-r border-gray-300 last:border-r-0 relative cursor-pointer"
          @click="jumpToStep(index)"
        >
          <div 
            class="absolute top-0 left-0 h-full bg-blue-500 transition-all duration-300 ease-in-out"
            :style="{ width: index <= currentIndex ? '100%' : '0%' }"
          ></div>
          <div class="absolute inset-0 flex items-center justify-center text-xs font-semibold" :class="{ 'text-white': index <= currentIndex, 'text-gray-700': index > currentIndex }">
            {{ item.stepLabel }}
          </div>
        </div>
      </div>
      <button 
        @click="navigateNext" 
        class="absolute right-0 top-0 bottom-0 w-6 bg-gray-300 hover:bg-gray-400 flex items-center justify-center z-10"
        :disabled="currentIndex >= totalSets - 1 || isAnimating"
      >
        &gt;
      </button>
    </div>
    <div class="flex-1 overflow-hidden">
      <transition name="fade" mode="out-in" @before-enter="lockNavigation" @after-leave="unlockNavigation">
        <Implications :key="currentIndex" :current-item="currentImplication" />
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import Applications from './Applications.vue'
import Implications from './Implications.vue'

const container = ref(null)
const scrollContainer = ref(null)
const currentIndex = ref(0)
const isAnimating = ref(false)
const wheelAccumulator = ref(0)
const WHEEL_THRESHOLD = 100 // Scroll Smoothness
const DEBOUNCE_DELAY = 50 // Scroll Delay

const timelineData = ref([
  { stepLabel: 'Step 1', application: { title: 'Application 1', description: 'Description of Application 1' },
    implication: { title: 'Implication 1', description: 'Description of Implication 1' } },
  { stepLabel: 'Step 2', application: { title: 'Application 2', description: 'Description of Application 2' },
    implication: { title: 'Implication 2', description: 'Description of Implication 2' } },
  { stepLabel: 'Step 3', application: { title: 'Application 3', description: 'Description of Application 3' },
    implication: { title: 'Implication 3', description: 'Description of Implication 3' } },
  { stepLabel: 'Step 4', application: { title: 'Application 4', description: 'Description of Application 4' },
    implication: { title: 'Implication 4', description: 'Description of Implication 4' } },
  { stepLabel: 'Step 5', application: { title: 'Application 5', description: 'Description of Application 5' },
    implication: { title: 'Implication 5', description: 'Description of Implication 5' } },
  { stepLabel: 'Step 6', application: { title: 'Application 6', description: 'Description of Application 6' },
    implication: { title: 'Implication 6', description: 'Description of Implication 6' } },
  { stepLabel: 'Step 7', application: { title: 'Application 7', description: 'Description of Application 7' },
    implication: { title: 'Implication 7', description: 'Description of Implication 7' } },
  { stepLabel: 'Step 8', application: { title: 'Application 8', description: 'Description of Application 8' },
    implication: { title: 'Implication 8', description: 'Description of Implication 8' } },
  { stepLabel: 'Step 9', application: { title: 'Application 9', description: 'Description of Application 9' },
    implication: { title: 'Implication 9', description: 'Description of Implication 9' } },
  { stepLabel: 'Step 10', application: { title: 'Application 10', description: 'Description of Application 10' },
    implication: { title: 'Implication 10', description: 'Description of Implication 10' } },
])

const totalSets = computed(() => timelineData.value.length)

const currentApplication = computed(() => timelineData.value[currentIndex.value].application)
const currentImplication = computed(() => timelineData.value[currentIndex.value].implication)

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
  transition: opacity 0.5s ease;  /* Transition Time */

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