<template>
  <button :disabled="props.disabled" class="border rounded-md h-10 text-sm px-3">{{ props.label }}</button>
</template>
<script setup>
const props = defineProps({
  label: {
    type: String,
    required: false,
    default: 'Submit',
  },
  disabled: {
    type: Boolean,
    required: false,
    default: false,
  },
}) 
</script>