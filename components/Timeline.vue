<template>
  <div
    class="min-h-screen flex flex-col bg-gradient-to-br from-sky-100 to-amber-100"
    @wheel.passive="handleWheel"
    tabindex="0"
    @keydown="handleKeyDown"
    ref="container"
  >
    <div
      ref="scrollContainer"
      class="h-12 md:h-8 bg-gradient-to-r from-sky-200 to-amber-200 relative flex items-center overflow-hidden"
    >
      <button
        @click="navigatePrev"
        class="absolute left-0 top-0 bottom-0 w-8 md:w-6 bg-sky-300 hover:bg-sky-400 flex items-center justify-center z-10 text-sky-900 dark:text-white text-lg md:text-base"
        :disabled="currentIndex <= 0 || isAnimating"
      >
        &lt;
      </button>
      <div class="absolute left-8 md:left-6 right-8 md:right-6 top-0 bottom-0 flex">
        <div
          v-for="(item, index) in limitedTimelineData"
          :key="index"
          class="flex-1 h-full border-r border-sky-300 last:border-r-0 relative cursor-pointer overflow-hidden"
          @click="jumpToStep(index)"
        >
          <div
            class="absolute inset-0 flex items-center justify-center text-xs font-semibold text-sky-900 z-10 transition-all duration-300"
            :class="{
              'text-sm md:text-base font-bold': index === currentIndex,
              'text-opacity-50': index !== currentIndex,
            }"
          >
            {{ item.stepLabel }}
          </div>
        </div>
      </div>
      <button
        @click="navigateNext"
        class="absolute right-0 top-0 bottom-0 w-8 md:w-6 bg-amber-300 hover:bg-amber-400 flex items-center justify-center z-10 text-amber-900 dark:text-white text-lg md:text-base"
        :disabled="currentIndex >= limitedTimelineData.length - 1 || isAnimating"
      >
        &gt;
      </button>
    </div>
    <div class="flex flex-col md:flex-1 overflow-hidden">
      <div class="min-h-[40vh] md:flex-1 overflow-hidden">
        <transition
          name="fade"
          mode="out-in"
          @before-enter="lockNavigation"
          @after-leave="unlockNavigation"
        >
          <Implications :key="currentIndex" :current-item="currentImplication" />
        </transition>
      </div>
      <div class="min-h-[40vh] md:flex-1 overflow-hidden">
        <transition
          name="fade"
          mode="out-in"
          @before-enter="lockNavigation"
          @after-leave="unlockNavigation"
        >
          <Applications :key="currentIndex" :current-item="currentApplication" />
        </transition>
      </div>
    </div>
    <footer class="flex items-center justify-center border-t">
      <a
        href="https://github.com/nathanbridgeearney/history-of-deepfakes/blob/main/README.md"
        target="_blank"
        rel="noopener noreferrer"
        class="flex items-center justify-center text-blue-600 hover:text-blue-800"
      >
        References Found Here
      </a>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";
import { useAsyncData } from "#app";
import Applications from "./Applications.vue";
import Implications from "./Implications.vue";

const container = ref(null);
const scrollContainer = ref(null);
const currentIndex = ref(0);
const isAnimating = ref(false);

const { data: timelineData } = await useAsyncData("timeline", () =>
  queryContent("/").find()
);

const limitedTimelineData = computed(() => {
  return timelineData.value
    .map((item) => ({
      stepLabel: item.stepLabel,
      application: {
        title: item.applicationTitle,
        description: item.applicationDescription.trim(),
      },
      implication: {
        title: item.implicationTitle,
        description: item.implicationDescription.trim(),
      },
    }))
    .slice(0, 6);
});

const totalSets = computed(() => limitedTimelineData.value.length);

const currentApplication = computed(
  () => limitedTimelineData.value[currentIndex.value].application
);
const currentImplication = computed(
  () => limitedTimelineData.value[currentIndex.value].implication
);

const handleKeyDown = (e) => {
  if (isAnimating.value) return;
  if (e.key === "ArrowLeft") {
    navigatePrev();
  } else if (e.key === "ArrowRight") {
    navigateNext();
  }
};

const setCurrentIndex = (newIndex) => {
  if (isAnimating.value) return;
  currentIndex.value = Math.max(
    0,
    Math.min(Math.round(newIndex), totalSets.value - 1)
  );
};

const navigatePrev = () => {
  if (isAnimating.value || currentIndex.value <= 0) return;
  setCurrentIndex(currentIndex.value - 1);
};

const navigateNext = () => {
  if (isAnimating.value || currentIndex.value >= totalSets.value - 1) return;
  setCurrentIndex(currentIndex.value + 1);
};

const jumpToStep = (index) => {
  if (isAnimating.value) return;
  setCurrentIndex(index);
};

const lockNavigation = () => {
  isAnimating.value = true;
};

const unlockNavigation = () => {
  isAnimating.value = false;
};

onMounted(() => {
  window.addEventListener("resize", handleResize);
  container.value.focus();
});

onUnmounted(() => {
  window.removeEventListener("resize", handleResize);
});

const handleResize = () => {
  // TODO
};
</script>

<style scoped>
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