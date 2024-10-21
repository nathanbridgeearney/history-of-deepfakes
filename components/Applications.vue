<template>
  <div class="h-full flex flex-col bg-sky-50 ">
    <h2 class="text-l font-bold p-2 bg-sky-100 text-black ">{{ currentItem.title }}</h2>
    <div class="flex-grow p-4 overflow-y-auto">
      <div class="bg-white shadow-md rounded-lg p-4">
        <div class="text-black">
          <p v-for="(paragraph, index) in paragraphs" :key="index" class="mb-4 text-sm text-justify	">{{ paragraph }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  currentItem: {
    type: Object,
    required: true
  }
});

const paragraphs = computed(() => {
  return props.currentItem.description.split('\n\n').filter(p => p.trim() !== '');
});
</script>