<template>
  <div class="h-full flex flex-col bg-sky-100 dark:bg-sky-950">
    <h2 class="text-2xl font-bold p-4 bg-sky-200 dark:bg-sky-900 text-black dark:text-black">Applications</h2>
    <div class="flex-grow p-4">
      <div class="bg-white dark:bg-gray-600 shadow-md rounded-lg p-4">
        <h3 class="text-lg font-semibold text-black dark:text-black">{{ currentItem.title }}</h3>
        <p class="mt-2 text-black dark:text-black">{{ currentItem.description }}</p>
      </div>
    </div>
  </div>
</template>
<script setup>
defineProps({
  currentItem: {
    type: Object,
    required: true
  }
})
</script>