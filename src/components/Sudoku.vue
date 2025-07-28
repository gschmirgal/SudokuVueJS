<template>
  <!-- Conteneur principal de la grille Sudoku -->
  <div class="sudoku-container">
    <h2>Sudoku Solver</h2>
    <!-- Grille Sudoku -->
    <table class="sudoku-grid">
      <tbody>
        <tr v-for="(row, rowIndex) in grid" :key="rowIndex">
          <td v-for="(cell, colIndex) in row" :key="colIndex" 
              :class="getCellClass(rowIndex, colIndex)">
            <!-- Champ de saisie pour chaque cellule -->
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
    <!-- Boutons d'action -->
    <div class="button-row">
      <button @click="solveGrid">Résoudre</button>
      <button @click="resetGrid">Réinitialiser</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

// Initialise une grille vide (9x9)
function getInitialGrid() {
  return Array(9).fill().map(() => Array(9).fill(""));
}
// Initialise le tableau de suivi des cellules saisies par l'utilisateur
function getInitialUserInput() {
  return Array(9).fill().map(() => Array(9).fill(false));
}

// Références réactives pour la grille et les entrées utilisateur
const grid = ref(getInitialGrid());
const userInput = ref(getInitialUserInput());

// Gère la saisie dans une cellule
function onInput(row, col) {
  const val = grid.value[row][col];
  // Vérifie que la valeur est un chiffre entre 1 et 9 ou vide
  if (!/^([1-9])?$/.test(val)) {
    grid.value[row][col] = "";
    userInput.value[row][col] = false;
  } else {
    userInput.value[row][col] = val !== "";
  }
}

// Réinitialise la grille et le suivi utilisateur
function resetGrid() {
  grid.value = getInitialGrid();
  userInput.value = getInitialUserInput();
}

// Retourne la classe CSS pour une cellule (bordures et couleur)
function getCellClass(row, col) {
  let className = '';
  // Bordures des blocs 3x3
  if(row % 3 === 2 && row !== 8) {
    className += ' block-border-bottom';
  }
  if(col % 3 === 2 && col !== 8) {
    className += ' block-border-right';
  }
  // Couleur selon origine du chiffre
  if (grid.value[row][col] !== "") {
    className += userInput.value[row][col] ? ' user-cell' : ' solved-cell';
  }
  return className.trim();
}

// Résout la grille de Sudoku
function solveGrid() {
  // Copie profonde de la grille sous forme de nombres
  const gridCopy = grid.value.map(row => row.map(cell => cell === "" ? 0 : parseInt(cell)));

  // Vérifie si un chiffre peut être placé à une position donnée
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

  // Vérifie la validité globale de la grille (pas de doublons)
  function isGridValid(grid) {
    // Vérifie lignes et colonnes
    for (let i = 0; i < 9; i++) {
      let rowSet = new Set();
      let colSet = new Set();
      for (let j = 0; j < 9; j++) {
        // Ligne
        if (grid[i][j] !== 0) {
          if (rowSet.has(grid[i][j]))
            return false;
          rowSet.add(grid[i][j]);
        }
        // Colonne
        if (grid[j][i] !== 0) {
          if (colSet.has(grid[j][i])) 
            return false;
          colSet.add(grid[j][i]);
        }
      }
    }
    // Blocs 3x3
    for (let blockRow = 0; blockRow < 3; blockRow++) {
      for (let blockCol = 0; blockCol < 3; blockCol++) {
        let blockSet = new Set();
        for (let i = 0; i < 3; i++) {
          for (let j = 0; j < 3; j++) {
            let val = grid[blockRow * 3 + i][blockCol * 3 + j];
            if (val !== 0) {
              if (blockSet.has(val)) 
                return false;
              blockSet.add(val);
            }
          }
        }
      }
    }
    return true;
  }

  // Si la grille n'est pas valide, affiche une erreur
  if (!isGridValid(gridCopy)) {
    alert("La grille n'est pas valide : il y a des doublons dans une ligne, colonne ou bloc.");
    return;
  }

  // Algorithme de backtracking pour résoudre la grille
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
          return false;
        }
      }
    }
    return true;
  }

  // Si une solution est trouvée, met à jour la grille
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

<style>
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
  color: black !important; /* chiffre saisi par l'utilisateur */
}
.solved-cell input {
  color: #22c55e !important; /* chiffre généré par le solveur */
}
/* Boutons côte à côte */
.button-row {
  display: flex;
  gap: 12px;
  margin-top: 16px;
}
button {
  margin-top: 0;
}
</style>
