<template>
    <div class="flex items-center space-x-2">
        <input :id="props.label.toLowerCase()"
            type="checkbox"
            class="h-3 w-3 text-primary border border-[#002C77] rounded-sm"
            :checked="props.checked"
            @change="props.toggle"
        />
        <label :for="props.label.toLowerCase()" class="text-sm text-[#002C77]">{{ props.label }}</label>
    </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
    label: {
        type: String,
        required: true,
    },
    checked: {
        type: Boolean,
        required: true,
    },
    toggle: {
        type: Function,
        required: false,
    },
})
</script>