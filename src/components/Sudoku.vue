<template>
  <div class="sudoku-container">
    <h2>Sudoku Solver</h2>
    <table class="sudoku-grid">
      <tbody>
        <tr v-for="(row, rowIndex) in grid" :key="rowIndex">
          <td v-for="(cell, colIndex) in row" :key="colIndex" 
              :class="getCellClass(rowIndex, colIndex)">
            <input
              :name="`cell-${rowIndex}-${colIndex}`"
              type="text"
              maxlength="1"
              v-model="grid[rowIndex][colIndex]"
              @input="onInput(rowIndex, colIndex)"
            />
          </td>
        </tr>
      </tbody>
    </table>
    <button @click="solveGrid">Résoudre</button>
    <button @click="resetGrid">Réinitialiser</button>
  </div>
</template>

<script setup>
import { ref } from 'vue';

function getInitialGrid() {
  return Array(9).fill().map(() => Array(9).fill(""));
}
function getInitialUserInput() {
  return Array(9).fill().map(() => Array(9).fill(false));
}

const grid = ref(getInitialGrid());
const userInput = ref(getInitialUserInput());

function onInput(row, col) {
  const val = grid.value[row][col];
  if (!/^([1-9])?$/.test(val)) {
    grid.value[row][col] = "";
    userInput.value[row][col] = false;
  } else {
    userInput.value[row][col] = val !== "";
  }
}

function resetGrid() {
  grid.value = getInitialGrid();
  userInput.value = getInitialUserInput();
}

function getCellClass(row, col) {
  let className = '';
  if(row % 3 === 2 && row !== 8) {
    className += ' block-border-bottom';
  }
  if(col % 3 === 2 && col !== 8) {
    className += ' block-border-right';
  }
  if (grid.value[row][col] !== "") {
    className += userInput.value[row][col] ? ' user-cell' : ' solved-cell';
  }
  return className.trim();
}

function solveGrid() {
  const gridCopy = grid.value.map(row => row.map(cell => cell === "" ? 0 : parseInt(cell)));

  function isValid(grid, row, col, num) {
    for (let x = 0; x < 9; x++) {
      if (grid[row][x] === num || grid[x][col] === num) 
        return false;
    }
    const startRow = Math.floor(row / 3) * 3;
    const startCol = Math.floor(col / 3) * 3;
    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        if (grid[startRow + i][startCol + j] === num) 
            return false;
      }
    }
    return true;
  }

  function solve(grid) {
    for (let row = 0; row < 9; row++) {
      for (let col = 0; col < 9; col++) {
        if (grid[row][col] === 0) {
          for (let num = 1; num <= 9; num++) {
            if (isValid(grid, row, col, num)) {
              grid[row][col] = num;
              if (solve(grid)) 
                return true;
              grid[row][col] = 0;
            }
          }
          console.log("erreur");
          return false;
        }
      }
    }
    return true;
  }

  if (solve(gridCopy)) {
    for (let row = 0; row < 9; row++) {
      for (let col = 0; col < 9; col++) {
        if (!userInput.value[row][col]) {
          grid.value[row][col] = gridCopy[row][col] === 0 ? "" : gridCopy[row][col].toString();
        }
      }
    }
  } else {
    alert("Aucune solution trouvée pour cette grille.");
  }
}
</script>

<style scoped>
.sudoku-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.sudoku-grid {
  border-collapse: collapse;
}
.sudoku-grid td {
  border: 1px solid #888;
  width: 40px;
  height: 40px;
  padding: 0;
}
.sudoku-grid input {
  width: 100%;
  height: 100%;
  text-align: center;
  font-size: 1.2em;
  border: none;
  outline: none;
}
.block-border-right {
  border-right: 2px solid #222 !important;
}
.block-border-bottom {
  border-bottom: 2px solid #222 !important;
}
.user-cell input {
  color: black !important; /* vert */
}
.solved-cell input {
  color: #22c55e !important; /* bleu */
}
button {
  margin-top: 16px;
}
</style>
