<template>
  <button
    :disabled="field.read_only"
    class="w-full px-4 py-2 bg-secondary text-white rounded-md  focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
    @click="$emit('click')"
  >
    {{ field.label }}
  </button>
</template>

<script setup>
defineProps({
  field: {
    type: Object,
    required: true
  }
})

defineEmits(['click'])
</script>


