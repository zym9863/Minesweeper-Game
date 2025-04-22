<script setup lang="ts">
import { computed } from 'vue';
import MineIcon from '../assets/icons/mine.svg';
import FlagIcon from '../assets/icons/flag.svg';

interface CellProps {
  row: number;
  col: number;
  isRevealed: boolean;
  isMine: boolean;
  isFlagged: boolean;
  adjacentMines: number;
  isGameOver: boolean;
}

const props = defineProps<CellProps>();
const emit = defineEmits(['reveal', 'flag']);

const handleLeftClick = () => {
  if (!props.isRevealed && !props.isFlagged && !props.isGameOver) {
    emit('reveal', { row: props.row, col: props.col });
  }
};

const handleRightClick = (event: MouseEvent) => {
  event.preventDefault();
  if (!props.isRevealed && !props.isGameOver) {
    emit('flag', { row: props.row, col: props.col });
  }
};

const showMine = computed(() => props.isRevealed && props.isMine);
const showFlag = computed(() => props.isFlagged);
const showNumber = computed(() => props.isRevealed && !props.isMine && props.adjacentMines > 0);

const cellClass = computed(() => {
  return {
    'cell': true,
    'revealed': props.isRevealed,
    'mine': props.isRevealed && props.isMine,
    'flagged': props.isFlagged,
    'empty': props.isRevealed && !props.isMine && props.adjacentMines === 0,
    [`adjacent-${props.adjacentMines}`]: props.isRevealed && !props.isMine && props.adjacentMines > 0,
    'pop': props.isRevealed && !props.isMine,
    'shake': props.isRevealed && props.isMine
  };
});
</script>

<template>
  <div
    :class="cellClass"
    @click="handleLeftClick"
    @contextmenu="handleRightClick"
  >
    <img v-if="showMine" :src="MineIcon" alt="Mine" class="cell-icon" />
    <img v-else-if="showFlag" :src="FlagIcon" alt="Flag" class="cell-icon" />
    <span v-else-if="showNumber" class="number">{{ props.adjacentMines }}</span>
  </div>
</template>

<style scoped>
.cell {
  width: 32px;
  height: 32px;
  border: 1px solid var(--board-border);
  background-color: var(--cell-unrevealed);
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: bold;
  user-select: none;
  cursor: pointer;
  position: relative;
  box-shadow: inset 2px 2px 0 rgba(255,255,255,0.2),
              inset -2px -2px 0 rgba(0,0,0,0.1);
  transition: all 0.1s ease;
}

.cell:hover:not(.revealed) {
  background-color: var(--cell-unrevealed-hover);
}

.revealed {
  background-color: var(--cell-revealed);
  box-shadow: none;
  border-color: var(--ui-border);
}

.mine {
  background-color: var(--cell-mine);
}

.flagged {
  background-color: var(--cell-flagged);
}

.empty {
  background-color: var(--cell-revealed);
}

.cell-icon {
  width: 20px;
  height: 20px;
  object-fit: contain;
}

.number {
  font-family: 'VT323', monospace;
  font-size: 20px;
  font-weight: bold;
}

.adjacent-1 { color: #2196F3; } /* Blue */
.adjacent-2 { color: #4CAF50; } /* Green */
.adjacent-3 { color: #F44336; } /* Red */
.adjacent-4 { color: #3F51B5; } /* Dark Blue */
.adjacent-5 { color: #9C27B0; } /* Purple */
.adjacent-6 { color: #009688; } /* Teal */
.adjacent-7 { color: #FF5722; } /* Deep Orange */
.adjacent-8 { color: #795548; } /* Brown */
</style>
