<template>
  <div class="h-full flex flex-col bg-green-50">
    <h2 class="text-2xl font-bold p-4 bg-green-100">Implications</h2>
    <div class="flex-grow p-4">
      <div class="bg-white shadow-md rounded-lg p-4">
        <h3 class="text-lg font-semibold">{{ currentItem.title }}</h3>
        <p class="mt-2 text-gray-600">{{ currentItem.description }}</p>
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