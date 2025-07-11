<template>

</template>

<script setup lang="ts">
interface Props { open: boolean }
const props = defineProps<Props>()
const emit = defineEmits(['close'])
</script>