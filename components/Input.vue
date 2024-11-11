<template>
    <div class="flex flex-col gap-2">
      <label :for="field.name" class="text-sm font-medium text-gray-700 dark:text-gray-200">{{ field.label }}</label>
      <input
        :id="field.name"
        :value="modelValue"
        @input="$emit('update:modelValue', $event.target.value)"
        :type="field.fieldtype === 'Password' ? 'password' : 'text'"
        :disabled="field.read_only"
        class="h-10 px-3 border rounded-md text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 dark:bg-gray-700 dark:text-gray-200 dark:border-gray-600"
      />
    </div>
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
  
  defineEmits(['update:modelValue'])
  </script>