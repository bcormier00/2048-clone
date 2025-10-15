<script setup>

import {computed, onBeforeUnmount, ref} from "vue";
import {onMounted} from "vue";

const gameArray = ref([
    [2, undefined, undefined, undefined],
    [2, undefined, undefined, undefined],
    [4, undefined, undefined, undefined],
    [undefined, undefined, undefined, undefined],

])

onMounted(() => {
    window.addEventListener("keydown", handleKeydown)
})

onBeforeUnmount(() => {
    window.removeEventListener("keydown", handleKeydown)
})

const flatGameArray = computed(() => gameArray.value.flat())


function handleKeydown(event) {
    if (["ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight"].includes(event.key)) {
        event.preventDefault();
    }
    if (event.key === "ArrowUp") {
        moveUp()
    }
    if (event.key === "ArrowDown") {
        moveDown()
    }
    if (event.key === "ArrowLeft") {
        moveLeft()
    }
    if (event.key === "ArrowRight") {
        moveRight()
    }

    spawnTile()
}

function spawnTile() {
    const emptyCells = [];

    for (let row = 0; row < 4; row++) {
        for (let col = 0; col < 4; col++) {
            if (gameArray.value[row][col] === undefined) {
                emptyCells.push([row, col]);
            }
        }
    }

    if (emptyCells.length === 0) return; // game over

    const [row, col] = emptyCells[Math.floor(Math.random() * emptyCells.length)];
    gameArray.value[row][col] = getNewCell();
}

function getNewCell() {
    return Math.random() < 0.9 ? 2 : 4;
}
function moveUp() {
    let alreadyMergedColumn = Array.from({ length: 4 }, () => Array(4).fill(false));

    gameArray.value.forEach((row, rowIndex) => {
        row.forEach((cell, colIndex) => {
            if(cell && rowIndex !== 0) { //dont care about the first row when moving up
                for(let i = rowIndex - 1; i >= 0; i--) { //find next available cell for the current one to move/combine with
                    if (gameArray.value[i][colIndex] === undefined) {
                        gameArray.value[i][colIndex] = cell // if i cell is undefined, current cell takes that spot
                        gameArray.value[i+1][colIndex] = undefined
                        continue
                    }
                    if (gameArray.value[i][colIndex] === cell && !alreadyMergedColumn[i][colIndex]) {
                        gameArray.value[i][colIndex] = cell + cell //if i cell is the same as cell, combine
                        gameArray.value[i+1][colIndex] = undefined
                        alreadyMergedColumn[i][colIndex] = true
                        break
                    }
                    if(gameArray.value[i][colIndex]) {
                        break //if i cell is populated, do nothing
                    }
                }
            }
        })
    })
}
function moveDown() {
    let alreadyMergedColumn = Array.from({ length: 4 }, () => Array(4).fill(false));

    for(let rowIndex = 2; rowIndex >= 0; rowIndex--) {
        for(let colIndex = 3; colIndex >= 0; colIndex--) {
            let cell = gameArray.value[rowIndex][colIndex]
            if(cell && rowIndex !== 3) {
                for(let i = rowIndex + 1; i <= 3; i++) {
                    if (gameArray.value[i][colIndex] === undefined) {
                        gameArray.value[i][colIndex] = cell
                        gameArray.value[i-1][colIndex] = undefined
                        continue
                    }
                    if (gameArray.value[i][colIndex] === cell && !alreadyMergedColumn[i][colIndex]) {
                        gameArray.value[i][colIndex] = cell + cell
                        gameArray.value[i-1][colIndex] = undefined
                        alreadyMergedColumn[i][colIndex] = true
                        break
                    }
                    if(gameArray.value[i][colIndex]) {
                        break
                    }
                }
            }
        }
    }
}

function moveLeft() {
    let alreadyMergedRow = Array.from({ length: 4 }, () => Array(4).fill(false));

    gameArray.value.forEach((row, rowIndex) => {
        row.forEach((cell, colIndex) => {
            if (cell && colIndex !== 0) {
                for (let i = colIndex - 1; i >= 0; i--) {
                    if (gameArray.value[rowIndex][i] === undefined) {
                        gameArray.value[rowIndex][i] = cell;
                        gameArray.value[rowIndex][i + 1] = undefined;
                        continue;
                    }
                    if (gameArray.value[rowIndex][i] === cell && !alreadyMergedRow[rowIndex][i]) {
                        gameArray.value[rowIndex][i] = cell + cell;
                        gameArray.value[rowIndex][i + 1] = undefined;
                        alreadyMergedRow[rowIndex][i] = true;
                        break;
                    }
                    if (gameArray.value[rowIndex][i]) {
                        break;
                    }
                }
            }
        });
    });
}

function moveRight() {
    let alreadyMergedRow = Array.from({ length: 4 }, () => Array(4).fill(false));

    for (let rowIndex = 0; rowIndex <= 3; rowIndex++) {
        for (let colIndex = 2; colIndex >= 0; colIndex--) {
            let cell = gameArray.value[rowIndex][colIndex];
            if (cell && colIndex !== 3) {
                for (let i = colIndex + 1; i <= 3; i++) {
                    if (gameArray.value[rowIndex][i] === undefined) {
                        gameArray.value[rowIndex][i] = cell;
                        gameArray.value[rowIndex][i - 1] = undefined;
                        continue;
                    }
                    if (gameArray.value[rowIndex][i] === cell && !alreadyMergedRow[rowIndex][i]) {
                        gameArray.value[rowIndex][i] = cell + cell;
                        gameArray.value[rowIndex][i - 1] = undefined;
                        alreadyMergedRow[rowIndex][i] = true;
                        break;
                    }
                    if (gameArray.value[rowIndex][i]) {
                        break;
                    }
                }
            }
        }
    }
}


</script>

<template>
    <div class="flex items-center justify-center min-h-screen">
        <div class="grid grid-cols-4 grid-rows-4 border-2 w-[500px] h-[500px]">
            <div
                v-for="(cell, i) in flatGameArray"
                :key="i"
                class="border-2 flex items-center justify-center font-bold"
            >
                {{ cell }}
            </div>
        </div>
    </div>
</template>

<style scoped>

</style>
