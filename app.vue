<template>
  <div class="tic-tac-toe">
    <h1>Tic Tac Toe</h1>
    <div class="board">
      <div
        v-for="(row, rowIndex) in board"
        :key="rowIndex"
        class="row"
      >
        <button
          v-for="(cell, cellIndex) in row"
          :key="cellIndex"
          :disabled="cell !== null || gameOver"
          class="cell"
          @click="makeMove(rowIndex, cellIndex)"
        >
          {{ cell }}
        </button>
      </div>
    </div>
    <p v-if="gameOver">
      {{ winner ? `${winner} wins!` : 'It\'s a draw!' }}
    </p>
    <div class="controls">
      <button
        class="reset"
        @click="resetGame"
      >
        Reset Game
      </button>
      <label>
        <input
          v-model="isAIOpponent"
          type="checkbox"
        > Play against AI
      </label>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

type Cell = 'X' | 'O' | null
type Board = Cell[][]
type Player = 'X' | 'O'

const board = ref<Board>([
  [null, null, null],
  [null, null, null],
  [null, null, null],
])

const currentPlayer = ref<Player>('X')
const isAIOpponent = ref<boolean>(false)
const gameOver = ref<boolean>(false)
const winner = ref<Player | null>(null)

const makeAIMove = (): void => {
  const emptyCells = board.value.flatMap((row, rowIndex) =>
    row.map((cell, colIndex) => ({ row: rowIndex, col: colIndex, value: cell })),
  ).filter(cell => cell.value === null)

  if (emptyCells.length > 0) {
    const randomCell = emptyCells[Math.floor(Math.random() * emptyCells.length)]
    setTimeout(() => makeMove(randomCell.row, randomCell.col), 500)
  }
}

const makeMove = (row: number, col: number): void => {
  if (board.value[row][col] === null && !gameOver.value) {
    board.value[row][col] = currentPlayer.value
    checkWinner()
    if (!gameOver.value) {
      currentPlayer.value = currentPlayer.value === 'X' ? 'O' : 'X'
      if (isAIOpponent.value && currentPlayer.value === 'O') {
        makeAIMove()
      }
    }
  }
}

const checkWinner = (): void => {
  const winPatterns: number[][][] = [
    [[0, 0], [0, 1], [0, 2]],
    [[1, 0], [1, 1], [1, 2]],
    [[2, 0], [2, 1], [2, 2]],
    [[0, 0], [1, 0], [2, 0]],
    [[0, 1], [1, 1], [2, 1]],
    [[0, 2], [1, 2], [2, 2]],
    [[0, 0], [1, 1], [2, 2]],
    [[0, 2], [1, 1], [2, 0]],
  ]

  for (const pattern of winPatterns) {
    const [a, b, c] = pattern
    if (
      board.value[a[0]][a[1]]
      && board.value[a[0]][a[1]] === board.value[b[0]][b[1]]
      && board.value[a[0]][a[1]] === board.value[c[0]][c[1]]
    ) {
      winner.value = board.value[a[0]][a[1]] as Player
      gameOver.value = true
      return
    }
  }

  if (board.value.flat().every(cell => cell !== null)) {
    gameOver.value = true
  }
}

const resetGame = (): void => {
  board.value = [
    [null, null, null],
    [null, null, null],
    [null, null, null],
  ]
  currentPlayer.value = 'X'
  gameOver.value = false
  winner.value = null
}
</script>

<style lang="scss">
.tic-tac-toe {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: Arial, sans-serif;

  .board {
    display: flex;
    flex-direction: column;
    margin: 20px 0;

    .row {
      display: flex;

      .cell {
        width: 60px;
        height: 60px;
        font-size: 24px;
        font-weight: bold;
        margin: 2px;
        cursor: pointer;

        &:disabled {
          cursor: not-allowed;
        }
      }
    }
  }

  .reset {
    margin-top: 10px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
  }

  .controls {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 20px;

    label {
      margin-top: 10px;
    }
  }
}
</style>
