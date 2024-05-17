<script setup lang="ts">
interface BlockState {
  x: number
  y: number
  adjacentMines: number
  revealed: boolean
  mine?: boolean
  flagged?: boolean
}

const WIDTH = 5
const HEIGHT = 5

const state = reactive(Array.from({ length: HEIGHT }, (_, y) => Array.from({ length: WIDTH }, (_, x): BlockState => ({
  x,
  y,
  revealed: false,
  adjacentMines: 0,
}))))
// 生成雷
function generateMines(initial: BlockState) {
  for (const row of state) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) <= 1)
        continue
      if (Math.abs(initial.y - block.y) <= 1)
        continue

      block.mine = Math.random() < 0.3
    }
  }

  updateNumbers()
}

const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, 1],
  [0, -1],
  [-1, 1],
  [-1, 0],
  [-1, -1],
]

const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-gray-500',
]
// 计算邻居雷数
function updateNumbers() {
  state.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine)
        return

      getSliblings(block).forEach((b) => {
        if (b.mine)
          block.adjacentMines += 1
      })
    })
  })
}

// 获取邻居
function getSliblings(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const y2 = block.y + dy
    const x2 = block.x + dx
    if (y2 < 0 || y2 >= HEIGHT || x2 < 0 || x2 >= WIDTH)
      return void 0

    return state[y2][x2]
  }).filter(Boolean) as BlockState[]
}

function getBlockClass(block: BlockState) {
  if (block.flagged)
    return 'bg-gray-500/10'
  if (!block.revealed)
    return 'bg-gray-500/10 hover:bg-gray-500/20'

  return block.mine ? 'bg-red-500/50' : numberColors[block.adjacentMines]
}

// 过滤出0
function expendZero(block: BlockState) {
  if (block.adjacentMines)
    return

  getSliblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true
      expendZero(s)
    }
  })
}

let mineGenerated = false
const isDev = false

function onContextmenu(block: BlockState) {
  if (block.revealed)
    return
  block.flagged = !block.flagged
}

function onClick(block: BlockState) {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }
  block.revealed = true
  if (block.mine) {
    // Game over
    alert('Game over')
    return
  }

  expendZero(block)
}

watchEffect(checkGameState)

function checkGameState() {
  const blocks = state.flat()
  if (blocks.every(block => block.revealed || block.flagged)) {
    if (blocks.some(block => block.flagged && !block.mine))
      alert('You cheated!')
    else
      alert('You win!')
  }
}
</script>

<template>
  <div>
    <h1>Welcome to Vue Minsweeper</h1>
    <div>
      <div v-for="(row, y) in state" :key="y" flex="~" items-center justify-center>
        <button v-for="(block, x) in row" :key="x" :class="getBlockClass(block)" flex="~" m-0.5 h-10 w-10 items-center justify-center border vertical-top @click="onClick(block)" @contextmenu.prevent="onContextmenu(block)">
          <template v-if="block.flagged">
            <div i-mdi:flag text-red-500 />
          </template>
          <template v-else-if="block.revealed || isDev">
            <div v-if="block.mine" i-mdi:mine />
            <div v-else>
              {{ block.adjacentMines || '0' }}
            </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>
