<script setup>
import { ref, computed } from 'vue'
import Cell from './components/Cell.vue'

const board = ref(Array(9).fill(null))
const xIsNext = ref(true)
const winner = ref(null)
const winningLine = ref([])

const isDraw = computed(() => !winner.value && board.value.every(v => v))
const gameOver = computed(() => !!winner.value || isDraw.value)

const status = computed(() => {
  if (winner.value) return `Winner: ${winner.value}!`
  if (isDraw.value) return 'Draw!'
  return `Next Player: ${xIsNext.value ? 'X' : 'O'}`
})

const winOverlay = computed(() => {
  if (!winner.value || winningLine.value.length !== 3) return null
  const line = winningLine.value
  const pos = ['16.666%', '50%', '83.333%']
  const isRow =
    (line[0] === 0 && line[1] === 1 && line[2] === 2) ||
    (line[0] === 3 && line[1] === 4 && line[2] === 5) ||
    (line[0] === 6 && line[1] === 7 && line[2] === 8)
  const isCol =
    (line[0] === 0 && line[1] === 3 && line[2] === 6) ||
    (line[0] === 1 && line[1] === 4 && line[2] === 7) ||
    (line[0] === 2 && line[1] === 5 && line[2] === 8)
  if (isRow) {
    const rowIndex = line[0] / 3
    return { type: 'row', value: pos[rowIndex] }
  }
  if (isCol) {
    const colIndex = line[0] % 3
    return { type: 'col', value: pos[colIndex] }
  }
  if (line[0] === 0 && line[1] === 4 && line[2] === 8) {
    return { type: 'diag', value: 'main' }
  }
  return { type: 'diag', value: 'anti' }
})

function onCellClick({ index }) {
  if (board.value[index] || winner.value) return
  board.value[index] = xIsNext.value ? 'X' : 'O'
  const res = calculateWinner(board.value)
  if (res) {
    winner.value = res.player
    winningLine.value = res.line
  } else {
    xIsNext.value = !xIsNext.value
  }
}

function isWinningCell(index) {
  return winningLine.value.includes(index)
}

function restartGame() {
  board.value = Array(9).fill(null)
  xIsNext.value = true
  winner.value = null
  winningLine.value = []
}

function calculateWinner(sq) {
  const lines = [
    [0, 1, 2], [3, 4, 5], [6, 7, 8],
    [0, 3, 6], [1, 4, 7], [2, 5, 8],
    [0, 4, 8], [2, 4, 6],
  ]
  for (const [a, b, c] of lines) {
    if (sq[a] && sq[a] === sq[b] && sq[a] === sq[c]) {
      return { player: sq[a], line: [a, b, c] }
    }
  }
  return null
}
</script>

<template>
  <div class="min-h-screen bg-[#f0fdf4] flex items-center justify-center px-4">
    <div class="w-full max-w-md">
      <div class="text-center mb-6">
        <h1 class="text-2xl font-semibold text-[#1a4332]">Tris</h1>
        <p class="mt-2 text-[#1a4332]">{{ status }}</p>
      </div>

      <div class="relative grid grid-cols-3 w-[90vw] max-w-md mx-auto">
        <transition name="winline">
          <div v-if="winOverlay?.type === 'row'"
               class="absolute left-0 w-full border-t-[4px] border-[#1a4332] opacity-80"
               :style="{ top: winOverlay.value, transform: 'translateY(-50%)' }"></div>
        </transition>
        <transition name="winline">
          <div v-if="winOverlay?.type === 'col'"
               class="absolute top-0 h-full border-l-[4px] border-[#1a4332] opacity-80"
               :style="{ left: winOverlay.value, transform: 'translateX(-50%)' }"></div>
        </transition>
        <transition name="winline">
          <div v-if="winOverlay?.type === 'diag' && winOverlay.value === 'main'"
               class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[142%] border-t-[4px] border-[#1a4332] opacity-80 rotate-45"></div>
        </transition>
        <transition name="winline">
          <div v-if="winOverlay?.type === 'diag' && winOverlay.value === 'anti'"
               class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[142%] border-t-[4px] border-[#1a4332] opacity-80 -rotate-45"></div>
        </transition>
        <Cell
          v-for="(val, idx) in board"
          :key="idx"
          :index="idx"
          :value="val"
          :disabled="!!val || !!winner"
          :isWinning="isWinningCell(idx)"
          @click="onCellClick"
        />
      </div>

      <div class="mt-6 flex justify-center">
        <button
          @click="restartGame"
          class="px-4 py-2 rounded-md bg-emerald-600 text-white hover:bg-emerald-700 transition-colors"
        >
          Restart Game
        </button>
      </div>
    </div>
    <transition name="modal">
      <div v-if="winner || isDraw" class="fixed inset-0 bg-black/40 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="w-full max-w-sm rounded-xl bg-white shadow-lg border border-[#1a4332]/30">
          <div class="px-6 pt-6 text-center">
            <h2 class="text-xl font-semibold text-[#1a4332]">
              {{ winner ? `Winner: ${winner}!` : 'Draw!' }}
            </h2>
            <p class="mt-2 text-sm text-[#1a4332]/80">
              {{ winner ? 'Hai fatto tris!' : 'Partita bloccata, riproviamo.' }}
            </p>
          </div>
          <div class="px-6 pb-6 mt-6 flex justify-center">
            <button
              @click="restartGame"
              class="px-4 py-2 rounded-md bg-emerald-600 text-white hover:bg-emerald-700 transition-colors"
            >
              Restart
            </button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
.winline-enter-active, .winline-leave-active { transition: opacity 200ms ease; }
.winline-enter-from, .winline-leave-to { opacity: 0; }
.modal-enter-active, .modal-leave-active { transition: opacity 200ms ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; }
</style>
