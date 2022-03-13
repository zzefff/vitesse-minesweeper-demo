<script setup lang="ts">
import type { BlockState } from '~/types'
import { isDev } from '~/composables/'

defineProps<{ block: BlockState }>()

function getBlockClass(block: BlockState) {
  if (!block.revealed) return 'bg-red'
  return block.mine ? 'bg-red' : ''
}
</script>

<template>
  <button
    w-10
    h-10
    border
    hover:bg-gray
    :class="getBlockClass(block)"
  >
    <span v-if="block.flagged">
      🚩
    </span>
    <span v-else-if="block.revealed">
      {{ block.mine ? '💣' : (block.adjacentMines || '') }}
    </span>
    <span v-else-if="!block.revealed || isDev">
      {{ block.mine ? '💣' : block.adjacentMines }}
    </span>
  </button>
</template>
