<script setup>

import {computed, onBeforeUnmount, ref} from "vue";
import {onMounted} from "vue";

const gameArray = ref([
    [2, undefined, undefined, undefined],
    [2, undefined, undefined, undefined],
    [undefined, undefined, undefined, undefined],
    [undefined, undefined, undefined, undefined],
])

onMounted(() => {
    window.addEventListener("keydown", handleKeydown)
})

onBeforeUnmount(() => {
    window.removeEventListener("keydown", handleKeydown)
})

const flatGameArray = computed(() => gameArray.value.flat())
let tilesMoved = false


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
    if (tilesMoved) {
        spawnTile()
    }
    isGameOver()
}

function isGameOver() {
    gameArray.value.forEach((row, rowIndex) => {
        row.forEach((cell, colIndex) => {
            if(checkNeighbourCellsForMerge(rowIndex, colIndex)) {
                return true
            }
        })
    })
    return false
}

function checkNeighbourCellsForMerge(rowIndex, colIndex) {

    const canMoveUp = gameArray.value[rowIndex - 1][colIndex] && gameArray.value[rowIndex - 1][colIndex] === gameArray.value[rowIndex][colIndex]
    console.log(canMoveUp, 'can you move up?', rowIndex, colIndex)
    return canMoveUp
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
    tilesMoved = false;
}

function getNewCell() {
    return Math.random() < 0.9 ? 2 : 4;
}

function getCellColour(cellValue) {
    switch (cellValue) {
        case 2: return "bg-amber-500"
        case 4: return "bg-green-500"
        case 8: return "bg-red-500"
        case 16: return "bg-red-600"
        case 32: return "bg-red-600"
        case 64: return "bg-red-600"
        case 128: return "bg-red-600"
        case 256: return "bg-red-600"
        case 512: return "bg-red-600"
        case 1024: return "bg-red-600"
        case 2048: return "bg-red-600"
        default: return "bg-gray-400"
    }
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
                        tilesMoved = true
                        continue
                    }
                    if (gameArray.value[i][colIndex] === cell && !alreadyMergedColumn[i][colIndex]) {
                        gameArray.value[i][colIndex] = cell + cell //if i cell is the same as cell, combine
                        gameArray.value[i+1][colIndex] = undefined
                        alreadyMergedColumn[i][colIndex] = true
                        tilesMoved = true
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
                        tilesMoved = true
                        continue
                    }
                    if (gameArray.value[i][colIndex] === cell && !alreadyMergedColumn[i][colIndex]) {
                        gameArray.value[i][colIndex] = cell + cell
                        gameArray.value[i-1][colIndex] = undefined
                        alreadyMergedColumn[i][colIndex] = true
                        tilesMoved = true
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
                        tilesMoved = true
                        continue;
                    }
                    if (gameArray.value[rowIndex][i] === cell && !alreadyMergedRow[rowIndex][i]) {
                        gameArray.value[rowIndex][i] = cell + cell;
                        gameArray.value[rowIndex][i + 1] = undefined;
                        alreadyMergedRow[rowIndex][i] = true;
                        tilesMoved = true
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
                        tilesMoved = true
                        continue;
                    }
                    if (gameArray.value[rowIndex][i] === cell && !alreadyMergedRow[rowIndex][i]) {
                        gameArray.value[rowIndex][i] = cell + cell;
                        gameArray.value[rowIndex][i - 1] = undefined;
                        alreadyMergedRow[rowIndex][i] = true;
                        tilesMoved = true
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
    <div class="flex items-center justify-center min-h-screen bg-black">
        <div class="grid grid-cols-4 grid-rows-4 border-2  w-[500px] h-[500px]">
            <div
                v-for="(cell, i) in flatGameArray"
                :key="i"
                class="border-2 flex items-center justify-center font-bold "
                :class="getCellColour(cell)"
            >
                {{ cell }}
            </div>
        </div>
    </div>
</template>
