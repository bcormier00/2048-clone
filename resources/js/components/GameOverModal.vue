<template>
    <div class="fixed inset-0 z-50 flex items-center justify-center bg-black/40">
        <div class="flex flex-col items-center gap-2 p-4 rounded-lg bg-white">
            <div class="font-extrabold font-sans">You suck</div>
            <div>Score: {{ props.score }}</div>
            <div>Best: {{ props.highScore }}</div>
            <button @click="emit('closeModal')" class="cursor-pointer border-4 border-board rounded-2xl bg-board text-white h-full w-full ">Run it back?</button>
        </div>
    </div>
</template>

<script setup>
const emit = defineEmits(['closeModal'])

const props = defineProps({
    score: Number,
    highScore: Number,
})
</script>
