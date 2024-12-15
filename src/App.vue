<script setup lang="ts">
import { reactive, ref, watch } from "vue";
import a1 from "./assets/a1.jpg"
import a2 from "./assets/a2.gif"

// Reactive table state
const tableData = reactive({
  rows: [],
  columns: [],
  values: { },
});

const hoveredCell = ref<{ row: string; col: string } | null>(null);

// Fetch data from localStorage on load
const loadTableData = () => {
  const storedData = localStorage.getItem("tableData");
  if (storedData) {
    Object.assign(tableData, JSON.parse(storedData));
  }
};

// Save table data to localStorage whenever it changes
watch(
  tableData,
  () => {
    localStorage.setItem("tableData", JSON.stringify(tableData));
  },
  { deep: true }
);

// Add a new row or column
const newValue = ref("");
function addEntry() {
  const value = newValue.value.trim();
  if (!value) return;

  if (!tableData.rows.includes(value)) {
    tableData.rows.push(value);
    tableData.values[value] = {};
    tableData.columns.forEach((col) => {
      tableData.values[value][col] = 0;
    });
  }

  if (!tableData.columns.includes(value)) {
    tableData.columns.push(value);
    tableData.rows.forEach((row) => {
      tableData.values[row][value] = 0;
    });
  }

  newValue.value = "";
}

// Delete a row or column
function deleteEntry() {
  const value = newValue.value.trim();
  if (!value) return;

  // Delete row
  const rowIndex = tableData.rows.indexOf(value);
  if (rowIndex !== -1) {
    tableData.rows.splice(rowIndex, 1);
    delete tableData.values[value];
  }

  // Delete column
  const colIndex = tableData.columns.indexOf(value);
  if (colIndex !== -1) {
    tableData.columns.splice(colIndex, 1);
    tableData.rows.forEach((row) => {
      delete tableData.values[row][value];
    });
  }

  newValue.value = "";
}

// Increment or decrement cell value
const changeValue = (row: string, col: string, delta: number) => {
  if (row !== col) {
    tableData.values[row][col] += delta;
  }
};

loadTableData();
</script>

<template>
  <main>
    <section>
      <h1 style="color: rgb(154, 82, 221);font-size: 40px;">Alice's epic word count tracker</h1>

      <!-- Add or delete entry input -->
      <div class="input-container">
        <input
          v-model="newValue"
          @keyup.enter="addEntry"
          type="text"
          placeholder="Add or delete row/column"
        />
        <button @click="addEntry">Add Entry</button>
        <button class="delete-button" @click="deleteEntry">Delete Entry</button>
      </div>

      <!-- Table -->
      <div class="table-container">
        <table border="1">
          <thead>
            <tr>
              <th></th>
              <th
                v-for="col in tableData.columns"
                :key="col"
                :class="{ highlight: hoveredCell?.col === col }"
                style="color: white;"
              >
                {{ col }}
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="row in tableData.rows" :key="row">
              <td :class="{ highlight: hoveredCell?.row === row }" style="color: white;">{{ row }}</td>
              <td
                v-for="col in tableData.columns"
                :key="col"
                :class="{ diagonal: row === col }"
                @click="changeValue(row, col, 1)"
                @mouseover="hoveredCell = { row, col }"
                @mouseout="hoveredCell = null"
              >
                <div class="cell-container">
                  <span>
                    {{ tableData.values[row][col] }}
                  </span>
                  <button
                    class="decrement-button"
                    @click.stop="changeValue(row, col, -1)"
                    :disabled="row === col"
                  >
                    -
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>

        <div class="img-container">
          <img :src="a1" alt="" width="500">
        <img :src="a2" alt="" height="300">
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped>

main{
  width: 100%;
  display: flex;
  justify-content: center;
}

/* Section styling */
section {
  text-align: center;
  width: 90%;
  max-width: 800px;
}

/* Input container styling */
.input-container {
  margin-bottom: 1.5rem;
  display: flex;
  justify-content: center;
  gap: 1rem;
}

input {
  padding: 0.5rem 1rem;
  font-size: 1.2rem;
}

button {
  padding: 0.5rem 1rem;
  font-size: 1.2rem;
  border: none;
  cursor: pointer;
  border-radius: 4px;
}

button.delete-button {
  background-color: red;
  color: white;
}

button:hover {
  opacity: 0.9;
}

/* Table styling */
.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
  font-size: 1.2rem;
}

th,
td {
  padding: 1rem;
  text-align: center;
  position: relative;
  border: 1px solid #ddd;
}

/* Highlight row or column headers */
.highlight {
  background-color: #8507d9;
}

/* Diagonal cell style */
.diagonal {
  background-color: #f9f9f9;
  pointer-events: none;
  color: #aaa;
}

/* Decrement button style */
.cell-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  user-select: none;
}

.decrement-button {
  position: absolute;
  top: 0;
  right: 0;
  font-size: 0.8rem;
  color: red;
  background: none;
  border: none;
  cursor: pointer;
  z-index: 1;
}

.decrement-button:disabled {
  cursor: not-allowed;
  color: #ccc;
}

.img-container{
  display: flex;
  padding-top: 20px;
  
}
</style>
