<script setup>

const puzzle = defineModel()
</script>

<template>
  <div class="board">
    <div v-for="(row, rowIndex) in puzzle" :key="`row-${rowIndex}`" class="row">
      <input v-for="(cell, cellIndex) in row" :key="`cell-${rowIndex}-${cellIndex}`" class="cell" v-model="puzzle[rowIndex][cellIndex]" type="number" min="1" max="9" />
    </div>
  </div>
</template>



<style>
.board {
  display: flex;
  flex-wrap: wrap;
  width: 450px;
}

.row {
  display: flex;
}

.cell {
  width: 50px;
  height: 50px;
  border: 1px solid #000;
  text-align: center;
  font-size: 20px;
}
</style>