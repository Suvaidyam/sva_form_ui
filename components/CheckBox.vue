<template>
    <div class="flex items-center space-x-2">
      <input 
        :id="field.name"
        :checked="modelValue"
        @change="$emit('update:modelValue', $event.target.checked)"
        type="checkbox"
        class="h-4 w-4 text-blue-600 border-gray-300 rounded focus:ring-blue-500 dark:border-gray-600 dark:focus:ring-blue-600"
        :disabled="field.read_only"
      />
      <label :for="field.name" class="text-sm text-gray-700 dark:text-gray-200">{{ field.label }}</label>
    </div>
  </template>
  
  <script setup>
  defineProps({
    field: {
      type: Object,
      required: true
    },
    modelValue: {
      type: Boolean,
      required: true
    }
  })
  
  defineEmits(['update:modelValue'])
  </script>