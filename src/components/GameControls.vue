<script setup lang="ts">
import { ref, computed } from 'vue';
import SmileIcon from '../assets/icons/smile.svg';
import SadIcon from '../assets/icons/sad.svg';
import CoolIcon from '../assets/icons/cool.svg';
import FlagIcon from '../assets/icons/flag.svg';
import TimerIcon from '../assets/icons/timer.svg';

interface GameConfig {
  rows: number;
  cols: number;
  mines: number;
}

const props = defineProps<{
  remainingMines: number;
  gameTime: number;
  isGameActive: boolean;
  gameStatus: 'playing' | 'won' | 'lost';
}>();

const emit = defineEmits(['restart', 'changeDifficulty', 'customConfig']);

// Predefined difficulty levels
const difficulties = {
  beginner: { rows: 9, cols: 9, mines: 10 },
  intermediate: { rows: 16, cols: 16, mines: 40 },
  expert: { rows: 16, cols: 30, mines: 99 }
};

const selectedDifficulty = ref('beginner');
const customRows = ref(9);
const customCols = ref(9);
const customMines = ref(10);
const showCustomSettings = ref(false);

// Format time display (MM:SS)
const formattedTime = computed(() => {
  const minutes = Math.floor(props.gameTime / 60);
  const seconds = props.gameTime % 60;
  return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
});

// Handle difficulty change
const handleDifficultyChange = (difficulty: string) => {
  selectedDifficulty.value = difficulty;
  showCustomSettings.value = difficulty === 'custom';

  if (difficulty !== 'custom') {
    emit('changeDifficulty', difficulties[difficulty as keyof typeof difficulties]);
  }
};

// Apply custom settings
const applyCustomSettings = () => {
  // Calculate maximum mines based on board size (to prevent too many mines)
  const maxMines = customRows.value * customCols.value - 1;
  const safeCustomMines = Math.min(customMines.value, maxMines);

  const customConfig: GameConfig = {
    rows: customRows.value,
    cols: customCols.value,
    mines: safeCustomMines
  };

  emit('customConfig', customConfig);
};

// Restart game
const restartGame = () => {
  emit('restart');
};
</script>

<template>
  <div class="game-controls">
    <div class="game-info">
      <div class="mine-counter">
        <img :src="FlagIcon" alt="Flag" class="counter-icon" />
        <span>{{ remainingMines }}</span>
      </div>
      <button class="restart-button" @click="restartGame">
        <img v-if="gameStatus === 'playing'" :src="SmileIcon" alt="Smile" class="face-icon" />
        <img v-else-if="gameStatus === 'won'" :src="CoolIcon" alt="Cool" class="face-icon" />
        <img v-else :src="SadIcon" alt="Sad" class="face-icon" />
      </button>
      <div class="timer">
        <img :src="TimerIcon" alt="Timer" class="counter-icon" />
        <span>{{ formattedTime }}</span>
      </div>
    </div>

    <div class="difficulty-selector">
      <div class="difficulty-options">
        <label>
          <input
            type="radio"
            name="difficulty"
            value="beginner"
            v-model="selectedDifficulty"
            @change="handleDifficultyChange('beginner')"
          >
          Beginner (9x9, 10 mines)
        </label>

        <label>
          <input
            type="radio"
            name="difficulty"
            value="intermediate"
            v-model="selectedDifficulty"
            @change="handleDifficultyChange('intermediate')"
          >
          Intermediate (16x16, 40 mines)
        </label>

        <label>
          <input
            type="radio"
            name="difficulty"
            value="expert"
            v-model="selectedDifficulty"
            @change="handleDifficultyChange('expert')"
          >
          Expert (16x30, 99 mines)
        </label>

        <label>
          <input
            type="radio"
            name="difficulty"
            value="custom"
            v-model="selectedDifficulty"
            @change="handleDifficultyChange('custom')"
          >
          Custom
        </label>
      </div>

      <div v-if="showCustomSettings" class="custom-settings">
        <div class="custom-input">
          <label for="custom-rows">Rows:</label>
          <input
            id="custom-rows"
            type="number"
            v-model.number="customRows"
            min="5"
            max="24"
          >
        </div>

        <div class="custom-input">
          <label for="custom-cols">Columns:</label>
          <input
            id="custom-cols"
            type="number"
            v-model.number="customCols"
            min="5"
            max="30"
          >
        </div>

        <div class="custom-input">
          <label for="custom-mines">Mines:</label>
          <input
            id="custom-mines"
            type="number"
            v-model.number="customMines"
            min="1"
            :max="customRows * customCols - 1"
          >
        </div>

        <button @click="applyCustomSettings">Apply</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-controls {
  margin-bottom: 20px;
}

.game-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: var(--board-bg);
  padding: 15px;
  border: 3px solid var(--board-border);
  margin-bottom: 15px;
  border-radius: 8px;
  box-shadow: 0 4px 8px var(--ui-shadow);
}

.mine-counter, .timer {
  font-family: 'VT323', monospace;
  font-size: 1.5em;
  font-weight: bold;
  background-color: #000;
  color: #f00;
  padding: 8px 12px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  gap: 8px;
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.5);
  border: 2px solid #333;
}

.counter-icon {
  width: 20px;
  height: 20px;
}

.face-icon {
  width: 30px;
  height: 30px;
}

.restart-button {
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  border-radius: 50%;
  border: 2px solid var(--board-border);
  background-color: var(--color-warning);
  padding: 0;
  box-shadow: 0 4px 8px var(--ui-shadow);
  transition: all 0.2s ease;
}

.restart-button:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 12px var(--ui-shadow);
}

.restart-button:active {
  transform: scale(0.95);
}

.difficulty-selector {
  margin-top: 15px;
  background-color: var(--ui-element-bg);
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 4px 8px var(--ui-shadow);
  border: 1px solid var(--ui-border);
}

.difficulty-options {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  margin-bottom: 15px;
}

.difficulty-options label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  padding: 8px 12px;
  border-radius: 4px;
  background-color: var(--ui-bg);
  border: 1px solid var(--ui-border);
  transition: all 0.2s ease;
}

.difficulty-options label:hover {
  background-color: var(--color-light);
  border-color: var(--color-primary);
}

.difficulty-options input[type="radio"] {
  accent-color: var(--color-primary);
  width: 16px;
  height: 16px;
}

.custom-settings {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  padding: 15px;
  background-color: var(--ui-bg);
  border-radius: 8px;
  align-items: center;
  border: 1px solid var(--ui-border);
  animation: fadeIn 0.3s;
}

.custom-input {
  display: flex;
  align-items: center;
  gap: 8px;
}

.custom-input label {
  font-weight: 500;
  color: var(--color-dark);
}

.custom-input input {
  width: 70px;
  padding: 8px;
  border: 1px solid var(--ui-border);
  border-radius: 4px;
  font-family: 'Roboto', sans-serif;
  transition: all 0.2s ease;
}

.custom-input input:focus {
  border-color: var(--color-primary);
  outline: none;
  box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.3);
}

button {
  padding: 8px 15px;
  background-color: var(--color-primary);
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s ease;
  box-shadow: 0 2px 4px var(--ui-shadow);
}

button:hover {
  background-color: #2980b9;
  transform: translateY(-1px);
  box-shadow: 0 4px 8px var(--ui-shadow);
}

button:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px var(--ui-shadow);
}
</style>
