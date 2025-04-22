<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue';
import GameBoard from './components/GameBoard.vue';
import GameControls from './components/GameControls.vue';

// Game state
const gameConfig = ref({
  rows: 9,
  cols: 9,
  mines: 10
});

const gameBoardRef = ref<InstanceType<typeof GameBoard> | null>(null);
const isGameActive = ref(true);
const gameTime = ref(0);
const flaggedCount = ref(0);
const timerInterval = ref<number | null>(null);
const gameStatus = ref<'playing' | 'won' | 'lost'>('playing');

// Computed properties
const remainingMines = computed(() => {
  return gameConfig.value.mines - flaggedCount.value;
});

// Methods
const startTimer = () => {
  // Clear any existing timer
  if (timerInterval.value) {
    clearInterval(timerInterval.value);
  }

  // Reset timer
  gameTime.value = 0;

  // Start new timer
  timerInterval.value = setInterval(() => {
    gameTime.value++;
  }, 1000) as unknown as number;
};

const stopTimer = () => {
  if (timerInterval.value) {
    clearInterval(timerInterval.value);
    timerInterval.value = null;
  }
};

const restartGame = () => {
  stopTimer();
  startTimer();
  isGameActive.value = true;
  gameStatus.value = 'playing';
  flaggedCount.value = 0;

  if (gameBoardRef.value) {
    gameBoardRef.value.resetGame();
  }
};

const handleGameOver = () => {
  stopTimer();
  isGameActive.value = false;
  gameStatus.value = 'lost';
};

const handleGameWon = () => {
  stopTimer();
  isGameActive.value = false;
  gameStatus.value = 'won';
};

const updateFlagCount = (count: number) => {
  flaggedCount.value = count;
};

const changeDifficulty = (config: typeof gameConfig.value) => {
  gameConfig.value = config;
  restartGame();
};

const applyCustomConfig = (config: typeof gameConfig.value) => {
  gameConfig.value = config;
  restartGame();
};

// Lifecycle hooks
onMounted(() => {
  startTimer();
});

onUnmounted(() => {
  stopTimer();
});
</script>

<template>
  <div class="minesweeper">
    <h1>Minesweeper</h1>

    <div v-if="gameStatus === 'won'" class="game-message win-message">
      Congratulations! You won! 🎉
    </div>

    <div v-if="gameStatus === 'lost'" class="game-message lose-message">
      Game Over! You hit a mine! 💥
    </div>

    <GameControls
      :remaining-mines="remainingMines"
      :game-time="gameTime"
      :is-game-active="isGameActive"
      :game-status="gameStatus"
      @restart="restartGame"
      @change-difficulty="changeDifficulty"
      @custom-config="applyCustomConfig"
    />

    <GameBoard
      ref="gameBoardRef"
      :config="gameConfig"
      @game-over="handleGameOver"
      @game-won="handleGameWon"
      @update-flag-count="updateFlagCount"
    />

    <div class="instructions">
      <h3>How to Play:</h3>
      <ul>
        <li>Left-click to reveal a cell</li>
        <li>Right-click to place/remove a flag</li>
        <li>Numbers show how many mines are adjacent to that cell</li>
        <li>Flag all mines to win!</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.minesweeper {
  font-family: 'Roboto', sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

h1 {
  font-family: 'VT323', monospace;
  font-size: 3.5em;
  color: var(--color-dark);
  margin-bottom: 20px;
  letter-spacing: 2px;
  text-transform: uppercase;
}

.game-message {
  margin: 15px 0;
  padding: 15px;
  border-radius: 8px;
  font-weight: bold;
  font-size: 1.2em;
  box-shadow: 0 4px 6px var(--ui-shadow);
  animation: pop 0.5s;
}

.win-message {
  background-color: var(--color-secondary);
  color: white;
  border: none;
}

.lose-message {
  background-color: var(--color-danger);
  color: white;
  border: none;
}

.instructions {
  margin-top: 30px;
  text-align: left;
  background-color: var(--ui-element-bg);
  padding: 20px;
  border-radius: 8px;
  border: 1px solid var(--ui-border);
  box-shadow: 0 4px 6px var(--ui-shadow);
}

.instructions h3 {
  margin-top: 0;
  color: var(--color-primary);
  font-weight: 600;
  border-bottom: 2px solid var(--ui-border);
  padding-bottom: 8px;
  margin-bottom: 15px;
}

.instructions ul {
  padding-left: 20px;
  line-height: 1.8;
}

.instructions li {
  margin-bottom: 8px;
}
</style>
