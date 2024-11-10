<template>
    <div class="flex flex-col gap-1.5">
        <label for="" class="text-sm">{{ field.label }}</label>
        <input class="border rounded-md p-2 outline-none" :disabled="field.read_only" :placeholder="placeholder" />
    </div>
</template>
<script setup>
const props = defineProps({
    field: {
        type: Object,
        required: false,
        default: {},
    },
})
</script>