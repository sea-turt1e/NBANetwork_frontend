<template>
    <div class="mb-4">
      <h3 class="text-lg font-semibold mb-2">選択された選手</h3>
      <div class="flex flex-wrap gap-2">
        <div
          v-for="player in selectedPlayers"
          :key="player.id"
          class="flex items-center bg-blue-100 rounded-full px-3 py-1"
        >
          <span>{{ player.name }}</span>
          <button
            @click="$emit('remove-player', player.id)"
            class="ml-2 text-red-500 hover:text-red-700"
          >
            ×
          </button>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      selectedPlayers: {
        type: Array,
        required: true
      }
    }
  }
  </script>