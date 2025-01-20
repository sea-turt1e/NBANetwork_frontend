Player Network Visualization App

<template>
  <div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">Player Network Visualizer</h1>
    
    <!-- 選手検索コンポーネント -->
    <div class="mb-6">
      <PlayerSearch 
        :players="availablePlayers"
        @player-selected="addSelectedPlayer"
      />
    </div>

    <!-- 選択された選手のリスト -->
    <div class="mb-6">
      <SelectedPlayers
        :selected-players="selectedPlayers"
        @remove-player="removePlayer"
      />
    </div>

    <!-- ネットワークグラフの描画エリア -->
    <NetworkGraph
      v-if="selectedPlayers.length > 0"
      :players="selectedPlayers"
      :relationships="networkData"
    />
  </div>
</template>

<script>
import { ref, watch } from 'vue';
import NetworkGraph from './components/NetworkGraph.vue';
import PlayerSearch from './components/PlayerSearch.vue';
import SelectedPlayers from './components/SelectedPlayers.vue';

export default {
  components: {
    PlayerSearch,
    SelectedPlayers,
    NetworkGraph
  },
  setup() {
    const availablePlayers = ref([])
    const selectedPlayers = ref([])
    const networkData = ref(null)

    // バックエンドから選手データを取得
    const fetchPlayers = async () => {
      try {
        const response = await fetch('/api/players')
        availablePlayers.value = await response.json()
      } catch (error) {
        console.error('Failed to fetch players:', error)
      }
    }

    // 選手間の関係データを取得
    const fetchNetworkData = async () => {
      if (selectedPlayers.value.length < 2) return
      
      try {
        const response = await fetch('/api/network', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            players: selectedPlayers.value.map(p => p.id)
          })
        })
        networkData.value = await response.json()
      } catch (error) {
        console.error('Failed to fetch network data:', error)
      }
    }

    // 選手の追加・削除の処理
    const addSelectedPlayer = (player) => {
      if (!selectedPlayers.value.find(p => p.id === player.id)) {
        selectedPlayers.value.push(player)
      }
    }

    const removePlayer = (playerId) => {
      selectedPlayers.value = selectedPlayers.value.filter(p => p.id !== playerId)
    }

    // 選手が変更されたらネットワークデータを再取得
    watch(selectedPlayers, () => {
      fetchNetworkData()
    })

    // コンポーネント初期化時に選手データを取得
    fetchPlayers()

    return {
      availablePlayers,
      selectedPlayers,
      networkData,
      addSelectedPlayer,
      removePlayer
    }
  }
}
</script>