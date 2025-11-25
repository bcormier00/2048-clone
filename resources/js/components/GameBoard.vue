<script setup>
/**
 * things to do:
 * Game over modal - make this not look like shit
 * gamestate in local storage
 * gamestate array as objects not numbers
 */

import { computed, onBeforeUnmount, ref, watch, onMounted } from 'vue'
import GameOverModal from '@/components/GameOverModal.vue'

onMounted(() => {
    window.addEventListener('keydown', handleKeydown)

    try {
        const highScoreFromStorage = localStorage.getItem('highScore')
        highScore.value = Number(highScoreFromStorage)

        const gameState = JSON.parse(localStorage.getItem('gameState'))

        if (gameState.board) {
            gameArray.value = gameState.board.map((row) =>
                row.map((cell) => (cell === null ? undefined : cell))
            )
            currentScore.value = gameState.score
        }
    } catch {
        startNewGame()
    }
})

onBeforeUnmount(() => {
    window.removeEventListener('keydown', handleKeydown)
})

// const gameOverScenario = [
//     [2, 4, 8, 2048],
//     [32, 8, 4, 128],
//     [128, 2, 8, 16],
//     [512, 1024, 32, 2048],
// ]

//     [2, undefined, undefined, undefined],
//     [undefined, undefined, undefined, undefined],
//     [undefined, undefined, undefined, undefined],
//     [undefined, undefined, undefined, undefined],

const gameArray = ref([
    [undefined, undefined, undefined, undefined],
    [undefined, undefined, undefined, undefined],
    [undefined, undefined, undefined, undefined],
    [undefined, undefined, undefined, undefined],
])

const flatGameArray = computed(() => gameArray.value.flat())

const isGameOver = computed(() => {
    return checkForNoAvailableMerges()
})

const currentScore = ref(0)
const highScore = ref(0)

let tilesMoved = false

watch(currentScore, (score) => {
    if (score > highScore.value) {
        highScore.value = score
        try {
            localStorage.setItem('highScore', String(score))
        } catch {}
    }
})

watch(
    [gameArray, currentScore],
    ([newGameArray, newScore]) => {
        const gameState = JSON.stringify({
            board: newGameArray,
            score: newScore,
        })

        try {
            localStorage.setItem('gameState', gameState)
        } catch {}
    },
    {
        deep: true,
    }
)

function handleKeydown(event) {
    if (
        ['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(event.key)
    ) {
        event.preventDefault()
    }
    if (event.key === 'ArrowUp') {
        moveUp()
    }
    if (event.key === 'ArrowDown') {
        moveDown()
    }
    if (event.key === 'ArrowLeft') {
        moveLeft()
    }
    if (event.key === 'ArrowRight') {
        moveRight()
    }
    if (tilesMoved) {
        spawnTile()
    }
}

function checkForNoAvailableMerges() {
    // grab last respective row/column depending on direction to check for merge (up would be bottom row - left would be rightmost column )
    //loop over each cell in said column/row and check if cell can merge in specified direction
    // move to next row/col in respect to the direction of the check (checking up merge = move to the next row above)
    //repeat step 2 and 3
    // do not need to check last row in respect to direction (up merge does not need to check top row)

    //check up merge
    for (let i = 3; i > 0; i--) {
        for (let j = 0; j < 4; j++) {
            const a = gameArray.value[i][j],
                b = gameArray.value[i - 1][j]
            // console.log(a, b)
            if (a === undefined || b === undefined || a === b) return false
        }
    }

    // DOWN: compare cell with one below; skip bottom row
    for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 4; j++) {
            const a = gameArray.value[i][j],
                b = gameArray.value[i + 1][j]
            if (a === undefined || b === undefined || a === b) return false
        }
    }

    // LEFT: compare cell with one to the left; skip leftmost col
    for (let i = 0; i < 4; i++) {
        for (let j = 3; j > 0; j--) {
            const a = gameArray.value[i][j],
                b = gameArray.value[i][j - 1]
            if (a === undefined || b === undefined || a === b) return false
        }
    }

    // RIGHT: compare cell with one to the right; skip rightmost col
    for (let i = 0; i < 4; i++) {
        for (let j = 0; j < 3; j++) {
            const a = gameArray.value[i][j],
                b = gameArray.value[i][j + 1]
            if (a === undefined || b === undefined || a === b) return false
        }
    }

    // no merges anywhere
    return true
}

function spawnTile() {
    const emptyCells = []

    for (let row = 0; row < 4; row++) {
        for (let col = 0; col < 4; col++) {
            if (gameArray.value[row][col] === undefined) {
                emptyCells.push([row, col])
            }
        }
    }

    if (emptyCells.length === 0) return // game over

    const [row, col] = emptyCells[Math.floor(Math.random() * emptyCells.length)]
    gameArray.value[row][col] = getNewCell()
    tilesMoved = false
}

function getNewCell() {
    return Math.random() < 0.9 ? 2 : 4
}

function getCellColour(cellValue) {
    let processedCellValue = cellValue
    if (cellValue > 2048) {
        processedCellValue = 3000
    }
    switch (processedCellValue) {
        case 2:
            return 'bg-2048-2'
        case 4:
            return 'bg-2048-4'
        case 8:
            return 'bg-2048-8'
        case 16:
            return 'bg-2048-16'
        case 32:
            return 'bg-2048-32'
        case 64:
            return 'bg-2048-64'
        case 128:
            return 'bg-2048-128'
        case 256:
            return 'bg-2048-256'
        case 512:
            return 'bg-2048-512'
        case 1024:
            return 'bg-2048-1024'
        case 2048:
            return 'bg-2048-2048'
        case 3000:
            return 'text-white bg-linear-to-tl from-violet-500 to-gold-500 to-red-500'
        default:
            return 'bg-empty-cell inset-shadow-black-2xs'
    }
}

const flipGameVertically = () => {
    gameArray.value = gameArray.value.reverse()
}

function flipGameHorizontally() {
    gameArray.value = gameArray.value.map((s) => s.reverse())
}

function moveUp() {
    let alreadyMergedColumn = Array.from({ length: 4 }, () =>
        Array(4).fill(false)
    )

    gameArray.value.forEach((row, rowIndex) => {
        row.forEach((cell, colIndex) => {
            if (cell && rowIndex !== 0) {
                //dont care about the first row when moving up
                for (let i = rowIndex - 1; i >= 0; i--) {
                    //find next available cell for the current one to move/combine with
                    if (gameArray.value[i][colIndex] === undefined) {
                        gameArray.value[i][colIndex] = cell // if i cell is undefined, current cell takes that spot
                        gameArray.value[i + 1][colIndex] = undefined
                        tilesMoved = true
                        continue
                    }
                    if (
                        gameArray.value[i][colIndex] === cell &&
                        !alreadyMergedColumn[i][colIndex]
                    ) {
                        gameArray.value[i][colIndex] = cell + cell //if i cell is the same as cell, combine
                        currentScore.value += cell + cell
                        gameArray.value[i + 1][colIndex] = undefined
                        alreadyMergedColumn[i][colIndex] = true
                        tilesMoved = true
                        break
                    }
                    if (gameArray.value[i][colIndex]) {
                        break //if i cell is populated, do nothing
                    }
                }
            }
        })
    })
}

function moveDown() {
    flipGameVertically()
    moveUp()
    flipGameVertically()
}

function moveLeft() {
    let alreadyMergedRow = Array.from({ length: 4 }, () => Array(4).fill(false))

    gameArray.value.forEach((row, rowIndex) => {
        row.forEach((cell, colIndex) => {
            if (cell && colIndex !== 0) {
                for (let i = colIndex - 1; i >= 0; i--) {
                    if (gameArray.value[rowIndex][i] === undefined) {
                        gameArray.value[rowIndex][i] = cell
                        gameArray.value[rowIndex][i + 1] = undefined
                        tilesMoved = true
                        continue
                    }
                    if (
                        gameArray.value[rowIndex][i] === cell &&
                        !alreadyMergedRow[rowIndex][i]
                    ) {
                        gameArray.value[rowIndex][i] = cell + cell
                        currentScore.value += cell + cell
                        gameArray.value[rowIndex][i + 1] = undefined
                        alreadyMergedRow[rowIndex][i] = true
                        tilesMoved = true
                        break
                    }
                    if (gameArray.value[rowIndex][i]) {
                        break
                    }
                }
            }
        })
    })
}

function moveRight() {
    flipGameHorizontally()
    moveLeft()
    flipGameHorizontally()
}

function startNewGame() {
    resetGameBoard()
    spawnTile()
}

function resetGameBoard() {
    gameArray.value = [
        [undefined, undefined, undefined, undefined],
        [undefined, undefined, undefined, undefined],
        [undefined, undefined, undefined, undefined],
        [undefined, undefined, undefined, undefined],
    ]
}
</script>

<template>
    <div
        class="flex flex-col items-center justify-center min-h-screen"
        :class="[isGameOver ? 'blur-sm' : 'blur-none']"
    >
        <div class="flex flex-col items-start space-y-1">
            <div class="flex w-full justify-between">
                <div class="text-2xl font-semibold">
                    Current Score: {{ currentScore }}
                </div>
                <div class="text-2xl font-semibold">
                    High Score: {{ highScore }}
                </div>
            </div>
            <div
                class="grid grid-cols-4 grid-rows-4 shadow-lg rounded-lg p-2 gap-2 w-[500px] h-[500px] bg-board"
            >
                <div
                    v-for="(cell, i) in flatGameArray"
                    :key="i"
                    class="flex items-center justify-center font-bold rounded-lg font-mono text-2xl text-black/70"
                    :class="getCellColour(cell)"
                >
                    {{ cell }}
                </div>
            </div>
        </div>
    </div>
    <game-over-modal
        v-if="isGameOver"
        :score="currentScore"
        :highScore="highScore"
        @closeModal="startNewGame"
    >
    </game-over-modal>
</template>
