<script setup>
const cell_value = defineModel()

</script>

<template>
  <input  class="cell" type="number" min="0" max="9" v-model="cell_value"/>
</template>