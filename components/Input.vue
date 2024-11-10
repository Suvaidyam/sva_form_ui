<template>
    <div class="flex flex-col gap-1.5">
        <label for="" class="text-sm">{{ label }}</label>
        <input class="border rounded-md p-2 outline-none" :disabled="disabled" :placeholder="placeholder" />
    </div>
</template>
<script setup>
const props = defineProps({
    label: {
        type: String,
        required: false,
        default: 'Name',
    },
    placeholder: {
        type: String,
        required: false,
    },
    disabled: {
        type: Boolean,
        required: false,
        default: false,
    },
})
</script>