<template>
  <button :disabled="read_only"
    class="border float-end rounded-md h-10 text-sm px-4 bg-blue-600 text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
    @click="$emit('click')">
    {{ field.label }}
  </button>
</template>

<script setup>
defineProps({
  field: {
    type: Object,
    required: true
  },
  modelValue: {
    type: String,
    required: true
  }

})

defineEmits(['click'])
</script>