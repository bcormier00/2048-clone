<template>
    <div class="fixed inset-0 z-50 flex items-center justify-center bg-black/40">
        <div class=" p-1.5 rounded-lg">you suck </div>
        <div class=" p-1.5 rounded-lg">your score {{props.score}}</div>
        <div class=" p-1.5 rounded-lg">best score {{ props.highScore}}</div>
        <button @click="emit('closeModal')" >Run it back?</button>
    </div>
</template>

<script setup>
const emit = defineEmits(['closeModal'])

const props = defineProps({
    score: Number,
    highScore: Number,
})
</script>
