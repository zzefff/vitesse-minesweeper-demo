<script setup lang="ts">

interface BlockState {
  x: number
  y: number
  mine: boolean
  revealed: boolean
  flagged: boolean
  adjacentMines: number
}

const WIDTH = 10
const HEIGHT = 10

const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
]

const state = reactive(Array.from({
  length: HEIGHT,
}, (_, y) => Array.from({ length: WIDTH }, (_, x): BlockState => {
  return {
    mine: false,
    adjacentMines: 0,
    revealed: false,
    flagged: false,
    x,
    y,
  }
})))

function getSiblings(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = dx + block.x
    const y2 = dy + block.y
    if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT) return undefined
    return state[y2][x2]
  }).filter(Boolean) as BlockState[]
}

function generateMines() {
  for (const row of state) {
    for (const block of row) {
      // if (Math.abs(initBlock.x - block.x) <= 1)
      //   continue
      // if (Math.abs(initBlock.y - block.y) <= 1)
      //   continue
      block.mine = Math.random() < 0.2
    }
  }
  updateNumbers()
}

function updateNumbers() {
  state.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine) return
      getSiblings(block).forEach((b) => {
        if (b.mine)
          block.adjacentMines += 1
      })
    })
  })
}

function expandZero(block: BlockState) {
  if (block.adjacentMines) return
  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true
      expandZero(s)
    }
  })
}

let mineGenerated = false
const dev = true

function getBlockClass(block: BlockState) {
  if (!block.revealed) return 'bg-red'
  return block.mine ? 'bg-red' : ''
}

if (!mineGenerated) {
  generateMines()
  mineGenerated = true
}

function onClick(block: BlockState) {
  block.revealed = true
  if (block.mine)
    alert('u dead mf!')
  expandZero(block)
}

</script>

<template>
  <div>
    Mine Sweeper

    <div
      v-for="(row, y) in state"
      :key="y"
      flex
      justify-center
      items-center
    >
      <button
        v-for="item in row"
        :key="item.x"
        w-10
        h-10
        border
        hover:bg-gray
        :class="getBlockClass(item)"
        @click="onClick(item)"
      >
        <span v-if="item.revealed">
          {{ item.mine ? '💣' : (item.adjacentMines || '') }}
        </span>
        <span v-else-if="!item.revealed || dev">
          {{ item.mine ? '💣' : item.adjacentMines }}
        </span>
      </button>
    </div>
  </div>
</template>
