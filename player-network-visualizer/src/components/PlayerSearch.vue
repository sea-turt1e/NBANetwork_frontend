<template>
    <div class="w-full">
      <input
        type="text"
        v-model="searchQuery"
        class="w-full p-2 border rounded"
        placeholder="選手名を検索..."
        @input="searchPlayers"
      />
      <div v-if="filteredPlayers.length > 0" class="mt-2">
        <div
          v-for="player in filteredPlayers"
          :key="player.id"
          class="cursor-pointer p-2 hover:bg-gray-100"
          @click="$emit('player-selected', player)"
        >
          {{ player.name }}
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import { computed, ref } from 'vue';
  
  export default {
    props: {
      players: {
        type: Array,
        required: true
      }
    },
    setup(props) {
      const searchQuery = ref('')
  
      const filteredPlayers = computed(() => {
        if (!searchQuery.value) return []
        return props.players.filter(player =>
          player.name.toLowerCase().includes(searchQuery.value.toLowerCase())
        )
      })
  
      return {
        searchQuery,
        filteredPlayers
      }
    }
  }
  </script>
