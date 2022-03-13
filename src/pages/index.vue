<script setup lang="ts">
import type { BlockState } from '~/types'
import { isDev, toggleDev } from '~/composables'

const WIDTH = 10
const HEIGHT = 10
const CHANCE = 0.2

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

const state = ref(Array.from({
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
    return state.value[y2][x2]
  }).filter(Boolean) as BlockState[]
}

function generateMines(initBlock: BlockState) {
  for (const row of state.value) {
    for (const block of row) {
      if (Math.abs(initBlock.x - block.x) <= 1)
        continue
      if (Math.abs(initBlock.y - block.y) <= 1)
        continue
      block.mine = Math.random() < CHANCE
    }
  }
  updateNumbers()
}

function updateNumbers() {
  state.value.forEach((row, y) => {
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

function onRightClick(block: BlockState) {
  if (block.revealed) return
  block.flagged = !block.flagged
}

function onClick(block: BlockState) {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }

  block.revealed = true
  if (block.mine)
    alert('u dead mf!')
  expandZero(block)
}

function checkState() {
  const blocks = state.value.flat()
  if (blocks.every(block => block.flagged || block.revealed)) {
    if (blocks.some(block => block.flagged && !block.mine))
      alert('fuck lose')
    else
      alert('not bad')
  }
}

watchEffect(checkState)

</script>

<template>
  <div>
    Mine Sweeper

    <button
      @click="toggleDev"
    >
      isDev: {{ isDev }}
    </button>

    <div
      v-for="(row, y) in state"
      :key="y"
      flex
      justify-center
      items-center
    >
      <Block
        v-for="item in row"
        :key="item.x"
        :block="item"
        @click="onClick(item)"
        @contextmenu.prevent="onRightClick(item)"
      />
    </div>
  </div>
</template>
