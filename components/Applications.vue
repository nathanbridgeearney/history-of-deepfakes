<template>
  <div class="h-full flex flex-col bg-sky-50 dark:bg-sky-950">
    <h2 class="text-1xl font-bold p-2 bg-sky-100 dark:bg-sky-900 text-black dark:text-black">{{ currentItem.title }}</h2>
    <div class="flex-grow p-4 overflow-y-auto">
      <div class="h-full bg-white dark:bg-gray-600 shadow-md rounded-lg p-4">
        <div class="text-black dark:text-black">
          <p v-for="(paragraph, index) in paragraphs" :key="index" class="mb-4">{{ paragraph }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>

const props = defineProps({
  currentItem: {
    type: Object,
    required: true
  }
})

const paragraphs = computed(() => {
  return props.currentItem.description.split('\n\n').filter(p => p.trim() !== '')
})
</script>