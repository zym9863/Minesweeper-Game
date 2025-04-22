<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue';
import Cell from './Cell.vue';

interface CellData {
  row: number;
  col: number;
  isMine: boolean;
  isRevealed: boolean;
  isFlagged: boolean;
  adjacentMines: number;
}

interface GameConfig {
  rows: number;
  cols: number;
  mines: number;
}

const props = defineProps<{
  config: GameConfig;
}>();

const emit = defineEmits(['gameOver', 'gameWon', 'updateFlagCount']);

const board = ref<CellData[][]>([]);
const isGameOver = ref(false);
const isFirstClick = ref(true);
const flaggedCount = ref(0);

// Initialize the game board
const initializeBoard = () => {
  isGameOver.value = false;
  isFirstClick.value = true;
  flaggedCount.value = 0;

  const newBoard: CellData[][] = [];

  // Create empty board
  for (let row = 0; row < props.config.rows; row++) {
    newBoard[row] = [];
    for (let col = 0; col < props.config.cols; col++) {
      newBoard[row][col] = {
        row,
        col,
        isMine: false,
        isRevealed: false,
        isFlagged: false,
        adjacentMines: 0
      };
    }
  }

  board.value = newBoard;
  emit('updateFlagCount', 0);
};

// Place mines randomly on the board, avoiding the first clicked cell
const placeMines = (firstClickRow: number, firstClickCol: number) => {
  let minesPlaced = 0;
  const totalCells = props.config.rows * props.config.cols;

  // Safety check to ensure we don't try to place more mines than cells
  const maxMines = Math.min(props.config.mines, totalCells - 1);

  while (minesPlaced < maxMines) {
    const row = Math.floor(Math.random() * props.config.rows);
    const col = Math.floor(Math.random() * props.config.cols);

    // Skip the first clicked cell and cells that already have mines
    if ((row === firstClickRow && col === firstClickCol) || board.value[row][col].isMine) {
      continue;
    }

    board.value[row][col].isMine = true;
    minesPlaced++;
  }

  // Calculate adjacent mines for each cell
  calculateAdjacentMines();
};

// Calculate the number of adjacent mines for each cell
const calculateAdjacentMines = () => {
  for (let row = 0; row < props.config.rows; row++) {
    for (let col = 0; col < props.config.cols; col++) {
      if (!board.value[row][col].isMine) {
        let count = 0;

        // Check all 8 adjacent cells
        for (let r = Math.max(0, row - 1); r <= Math.min(props.config.rows - 1, row + 1); r++) {
          for (let c = Math.max(0, col - 1); c <= Math.min(props.config.cols - 1, col + 1); c++) {
            if (!(r === row && c === col) && board.value[r][c].isMine) {
              count++;
            }
          }
        }

        board.value[row][col].adjacentMines = count;
      }
    }
  }
};

// Reveal a cell
const revealCell = ({ row, col }: { row: number; col: number }) => {
  if (isGameOver.value || board.value[row][col].isRevealed || board.value[row][col].isFlagged) {
    return;
  }

  // On first click, place mines (ensuring first click is never a mine)
  if (isFirstClick.value) {
    placeMines(row, col);
    isFirstClick.value = false;
  }

  // Reveal the cell
  board.value[row][col].isRevealed = true;

  // If it's a mine, game over
  if (board.value[row][col].isMine) {
    gameOver();
    return;
  }

  // If it's an empty cell (no adjacent mines), reveal adjacent cells recursively
  if (board.value[row][col].adjacentMines === 0) {
    revealAdjacentCells(row, col);
  }

  // Check if the game is won
  checkWinCondition();
};

// Recursively reveal adjacent cells for empty cells
const revealAdjacentCells = (row: number, col: number) => {
  for (let r = Math.max(0, row - 1); r <= Math.min(props.config.rows - 1, row + 1); r++) {
    for (let c = Math.max(0, col - 1); c <= Math.min(props.config.cols - 1, col + 1); c++) {
      const cell = board.value[r][c];

      // Skip if it's the same cell, already revealed, or flagged
      if ((r === row && c === col) || cell.isRevealed || cell.isFlagged) {
        continue;
      }

      // Reveal the cell
      cell.isRevealed = true;

      // If it's also an empty cell, recursively reveal its adjacent cells
      if (cell.adjacentMines === 0) {
        revealAdjacentCells(r, c);
      }
    }
  }
};

// Toggle flag on a cell
const toggleFlag = ({ row, col }: { row: number; col: number }) => {
  if (isGameOver.value || board.value[row][col].isRevealed) {
    return;
  }

  const cell = board.value[row][col];
  cell.isFlagged = !cell.isFlagged;

  // Update flag count
  flaggedCount.value += cell.isFlagged ? 1 : -1;
  emit('updateFlagCount', flaggedCount.value);

  // Check if the game is won
  checkWinCondition();
};

// Check if the game is won
const checkWinCondition = () => {
  // Game is won when all non-mine cells are revealed
  for (let row = 0; row < props.config.rows; row++) {
    for (let col = 0; col < props.config.cols; col++) {
      const cell = board.value[row][col];

      // If there's a non-mine cell that's not revealed, the game is not won yet
      if (!cell.isMine && !cell.isRevealed) {
        return;
      }
    }
  }

  // If we get here, all non-mine cells are revealed, so the game is won
  isGameOver.value = true;
  emit('gameWon');
};

// Game over function
const gameOver = () => {
  isGameOver.value = true;

  // Reveal all mines
  for (let row = 0; row < props.config.rows; row++) {
    for (let col = 0; col < props.config.cols; col++) {
      if (board.value[row][col].isMine) {
        board.value[row][col].isRevealed = true;
      }
    }
  }

  emit('gameOver');
};

// Reset the game
const resetGame = () => {
  initializeBoard();
};

// Watch for config changes to reset the game
watch(() => props.config, () => {
  resetGame();
}, { deep: true });

// Initialize the board when the component is mounted
onMounted(() => {
  initializeBoard();
});

// Expose resetGame method to parent component
defineExpose({ resetGame });
</script>

<template>
  <div class="game-board">
    <div v-for="(row, rowIndex) in board" :key="`row-${rowIndex}`" class="board-row">
      <Cell
        v-for="(cell, colIndex) in row"
        :key="`cell-${rowIndex}-${colIndex}`"
        :row="rowIndex"
        :col="colIndex"
        :is-revealed="cell.isRevealed"
        :is-mine="cell.isMine"
        :is-flagged="cell.isFlagged"
        :adjacent-mines="cell.adjacentMines"
        :is-game-over="isGameOver"
        @reveal="revealCell"
        @flag="toggleFlag"
      />
    </div>
  </div>
</template>

<style scoped>
.game-board {
  display: inline-block;
  border: 3px solid var(--board-border);
  background-color: var(--board-bg);
  padding: 8px;
  border-radius: 8px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

.board-row {
  display: flex;
  margin-bottom: 1px;
}

.board-row:last-child {
  margin-bottom: 0;
}
</style>
