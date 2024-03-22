<script setup>
import { ref } from 'vue'
import axios from "axios";
import SudokuMatrix from "./components/SudokuMatrix.vue";

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL

axios.defaults.withCredentials = true;

const puzzle = ref([
  ['', '', '', '', '', '', '', '', ''],
  ['', '', '', '', '', '', '', '', ''],
  ['', '', '', '', '', '', '', '', ''],
  ['', '', '', '', '', '', '', '', ''],
  ['', '', '', '', '', '', '', '', ''],
  ['', '', '', '', '', '', '', '', ''],
  ['', '', '', '', '', '', '', '', ''],
  ['', '', '', '', '', '', '', '', ''],
  ['', '', '', '', '', '', '', '', ''],
])
const imageFile = ref(null)
const uploadedImagePreview = ref(null)
const sudokuImagePreview = ref(null)
const shouldSudokuSolveResultDisplay = ref(false)
const isSudokuPuzzleSolved = ref(false)

// // this need to be done for each cell in the puzzle
// watch(puzzle, async (oldPuzzle, newPuzzle) => {
//   console.log(oldPuzzle)
//   console.log(newPuzzle)
//   if (oldPuzzle !== newPuzzle) {
//     shouldSudokuSolveResultDisplay.value = false
//   }
// })

const utilsIsImage = (file) => {
  try {
    return file.value.type.startsWith('image/');
  } catch (error) {
    console.log(errro);
    return false;
  }
}

const utilsCopyConvertToDisplay = (dataMatrix) => {
  const displayMatrix = dataMatrix.map((row) => {
    return row.slice()
  })
  displayMatrix.forEach((row, row_idx, m) => {
    row.forEach((ele, col_idx) => {
      if (ele === 0) {
        m[row_idx][col_idx] = '';
      }
    })
  });
  return displayMatrix;
}

const utilsCopyConvertToData = (displayMatrix) => {
  const dataMatrix = displayMatrix.map((row) => {
    return row.slice()
  })
  dataMatrix.forEach((row, row_idx, m) => {
    row.forEach((ele, col_idx) => {
      if (ele === '') {
        m[row_idx][col_idx] = 0;
      }
    })
  });
  return dataMatrix
}

const onChangeFileUpload = async (event) => {
  imageFile.value = event.target.files[0];
  if (imageFile.value === undefined) {
    sudokuImagePreview.value = null
    uploadedImagePreview.value = null
    isSudokuPuzzleSolved.value = false
    return
  }
  if (!utilsIsImage(imageFile)) {
    alert("Please upload an image file.")
    return
  }

  // show the uploaded image
  uploadedImagePreview.value = URL.createObjectURL(imageFile.value);

  // request for digit recognition
  const formData = new FormData();
  formData.append('sudoku_image_file', imageFile.value);

  try {
    const endpoint = API_BASE_URL + "/sudoku/load/image";
    const response = await axios.post(endpoint, formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    });
    // get matrix
    const dataMatrix = response.data.matrix;
    const displayMatrix = utilsCopyConvertToDisplay(dataMatrix);
    puzzle.value = displayMatrix;

    // get the extracted sudoku image
    const sudoku_image_jpg_binary_base64 = response.data.sudoku_image_jpg_binary_base64_utf8
    sudokuImagePreview.value = sudoku_image_jpg_binary_base64
  } catch (error) {
    console.error(error);
  }


};

const onClickSolveSudokuPuzzle = async () => {
  const dataMatrix = utilsCopyConvertToData(puzzle.value)
  try {
    const endpoint = API_BASE_URL + "/sudoku/solve";
    const response = await axios.post(endpoint, { matrix: dataMatrix }, {
      headers: {
        'Content-Type': 'application/json',
      }
    })
    if (response.data !== null) {
      shouldSudokuSolveResultDisplay.value = true;
    }
    isSudokuPuzzleSolved.value = response.data.is_solved;
    const matrix = response.data.matrix;
    if (isSudokuPuzzleSolved.value && matrix !== null) {
      puzzle.value = matrix;
    }
  } catch (error) {
    console.error();
  }
}

</script>

<template>
  <main>
    <h1>Sudoku Puzzle</h1>
    <SudokuMatrix v-model="puzzle" />

    <!-- Upload Sudoku Image -->
    <div>
      <label>File
        <input type="file" id="file" ref="imageFile" accept="image/*" v-on:change="onChangeFileUpload" />
      </label>
      <div></div>
    </div>

    <!-- Solve Button -->
    <div class="sudoku-solve-btn">
      <button @click="onClickSolveSudokuPuzzle">Solve Puzzle</button>
      <div v-if="shouldSudokuSolveResultDisplay">
        <span v-if="isSudokuPuzzleSolved">Puzzle is solved.</span>
        <span v-else>No solution exist!</span>
      </div>
    </div>

    <!-- Display Sudoku Image Preview-->
    <div class="sudoku-image-preview" v-if="sudokuImagePreview">
      <h2>Sudoku Image (Extracted from the uploaded image):</h2>
      <img v-if="utilsIsImage" :src="'data:image/jpeg;base64,' + sudokuImagePreview" alt="Sudoku Image Preview" class="img-thumbnail" />
    </div>

    <!-- Display Uploaded Image Preview-->
    <div class="uploaded-image-preview" v-if="uploadedImagePreview">
      <h2>Uploaded Image:</h2>
      <img v-if="utilsIsImage" :src="uploadedImagePreview" alt="Uploaded Image Preview" class="img-thumbnail" />
    </div>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}

.sudoku-image-preview img {
  width: 85%;
  height: auto;
  margin-left: 20px;
}

.uploaded-image-preview img {
  width: 70%;
  /* Adjust as needed */
  height: auto;
  margin-left: 20px;
}
</style>
