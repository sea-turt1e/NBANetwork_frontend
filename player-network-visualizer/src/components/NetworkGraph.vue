<template>
    <div class="relative">
      <div class="absolute top-2 right-2 flex gap-2">
        <button
          @click="zoomIn"
          class="bg-white p-2 rounded shadow hover:bg-gray-100"
        >
          <span>+</span>
        </button>
        <button
          @click="zoomOut"
          class="bg-white p-2 rounded shadow hover:bg-gray-100"
        >
          <span>-</span>
        </button>
      </div>
      <div class="w-full h-[600px]" ref="graphContainer"></div>
      
      <!-- ツールチップ -->
      <div
        v-if="hoveredNode"
        class="absolute bg-white p-2 rounded shadow-lg"
        :style="tooltipStyle"
      >
        <h4 class="font-bold">{{ hoveredNode.name }}</h4>
        <div v-if="hoveredNode.stats">
          <div>得点: {{ hoveredNode.stats.points }}</div>
          <div>アシスト: {{ hoveredNode.stats.assists }}</div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import * as d3 from 'd3';
import { computed, onMounted, ref, watch } from 'vue';
  
  export default {
    props: {
      players: {
        type: Array,
        required: true
      },
      relationships: {
        type: Object,
        required: true
      }
    },
    setup(props) {
      const graphContainer = ref(null)
      const hoveredNode = ref(null)
      const mousePosition = ref({ x: 0, y: 0 })
      let svg = null
      let zoom = null
  
      const tooltipStyle = computed(() => ({
        left: `${mousePosition.value.x + 10}px`,
        top: `${mousePosition.value.y + 10}px`
      }))
  
      const initializeGraph = () => {
        const width = graphContainer.value.clientWidth
        const height = 600
  
        // ズーム機能の設定
        zoom = d3.zoom()
          .scaleExtent([0.5, 2])
          .on('zoom', (event) => {
            svg.select('g').attr('transform', event.transform)
          })
  
        svg = d3.select(graphContainer.value)
          .append('svg')
          .attr('width', width)
          .attr('height', height)
          .call(zoom)
          .append('g')
  
        // カラースケールの定義
        const colorScale = d3.scaleOrdinal(d3.schemeCategory10)
  
        const simulation = d3.forceSimulation()
          .force('link', d3.forceLink().id(d => d.id).distance(100))
          .force('charge', d3.forceManyBody().strength(-200))
          .force('center', d3.forceCenter(width / 2, height / 2))
          .force('collision', d3.forceCollide().radius(30))
  
        const nodes = props.players.map(player => ({
          ...player,
          r: 20 // ノードの半径
        }))
  
        const links = props.relationships.map(rel => ({
          source: rel.player1Id,
          target: rel.player2Id,
          value: rel.weight
        }))
  
        // エッジの描画
        const link = svg.append('g')
          .selectAll('line')
          .data(links)
          .enter()
          .append('line')
          .attr('stroke-width', d => d.value * 3)
          .attr('stroke', '#999')
          .attr('opacity', 0.6)
  
        // エッジの値のラベル
        const linkLabels = svg.append('g')
          .selectAll('text')
          .data(links)
          .enter()
          .append('text')
          .attr('text-anchor', 'middle')
          .attr('dy', -5)
          .text(d => d.value.toFixed(3))
          .attr('font-size', '10px')
  
        // ノードグループの作成
        const node = svg.append('g')
          .selectAll('g')
          .data(nodes)
          .enter()
          .append('g')
          .call(d3.drag()
            .on('start', dragstarted)
            .on('drag', dragged)
            .on('end', dragended))
  
        // ノードの円を描画
        node.append('circle')
          .attr('r', d => d.r)
          .attr('fill', d => colorScale(d.id))
          .attr('stroke', '#fff')
          .attr('stroke-width', 2)
  
        // ノードのラベルを描画
        node.append('text')
          .text(d => d.name)
          .attr('text-anchor', 'middle')
          .attr('dy', '.35em')
          .attr('font-size', '12px')
          .attr('fill', '#fff')
  
        // ホバー効果の追加
        node
          .on('mouseover', (event, d) => {
            hoveredNode.value = d
            mousePosition.value = {
              x: event.pageX,
              y: event.pageY
            }
          })
          .on('mouseout', () => {
            hoveredNode.value = null
          })
  
        // シミュレーションの更新関数
        simulation
          .nodes(nodes)
          .on('tick', () => {
            link
              .attr('x1', d => d.source.x)
              .attr('y1', d => d.source.y)
              .attr('x2', d => d.target.x)
              .attr('y2', d => d.target.y)
  
            linkLabels
              .attr('x', d => (d.source.x + d.target.x) / 2)
              .attr('y', d => (d.source.y + d.target.y) / 2)
  
            node
              .attr('transform', d => `translate(${d.x},${d.y})`)
          })
  
        simulation.force('link').links(links)
  
        // ドラッグ関連の関数
        function dragstarted(event) {
          if (!event.active) simulation.alphaTarget(0.3).restart()
          event.subject.fx = event.subject.x
          event.subject.fy = event.subject.y
        }
  
        function dragged(event) {
          event.subject.fx = event.x
          event.subject.fy = event.y
        }
  
        function dragended(event) {
          if (!event.active) simulation.alphaTarget(0)
          event.subject.fx = null
          event.subject.fy = null
        }
      }
  
      // ズーム機能
      const zoomIn = () => {
        zoom.scaleBy(svg.transition().duration(750), 1.2)
      }
  
      const zoomOut = () => {
        zoom.scaleBy(svg.transition().duration(750), 0.8)
      }
  
      onMounted(() => {
        initializeGraph()
      })
  
      watch([() => props.players, () => props.relationships], () => {
        if (svg) {
          svg.selectAll('*').remove()
          initializeGraph()
        }
      })
  
      return {
        graphContainer,
        hoveredNode,
        tooltipStyle,
        zoomIn,
        zoomOut
      }
    }
  }
  </script>