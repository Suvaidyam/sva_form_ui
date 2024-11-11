<template>
  <div class="flex flex-col gap-2">
    <label class="text-sm font-medium text-gray-700 dark:text-gray-200">{{ field.label }}</label>
    <div class="space-y-2">
      <div v-for="option in options" :key="option.name" class="flex items-center">
        <input
          :id="`${field.name}-${option.name}`"
          :name="field.name"
          type="checkbox"
          :checked="isChecked(option)"
          @change="updateValue(option)"
          class="h-4 w-4 text-blue-600 border-gray-300 rounded focus:ring-blue-500 dark:border-gray-600 dark:focus:ring-blue-600"
        />
        <label :for="`${field.name}-${option.name}`" class="ml-2 block text-sm text-gray-700 dark:text-gray-200">
          {{ option.label }}
        </label>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, inject } from 'vue'

const props = defineProps({
  field: {
    type: Object,
    required: true
  },
  modelValue: {
    type: Array,
    required: true
  }
  
})

const emit = defineEmits(['update:modelValue'])

const call = inject('$call')

const options = ref([])

const getOptions = async () => {
  try {
    let filters = {}
    if (props.field.link_filters) {
      try {
        filters = JSON.parse(props.field.link_filters)
      } catch (e) {
        console.error('Invalid link_filters JSON:', e)
      }
    } else {
      filters = { field: props.field.fieldname }
    }
    const response = await call('frappe.client.get_list', { 
      doctype: 'Field Options',
      filters: filters,
      fields: ['*'],
    })
    options.value = response
  } catch (err) {
    console.error('Error fetching options:', err)
  }
}

const isChecked = (option) => {
  return props.modelValue.some(item => item.name === option.name)
}

const updateValue = (option) => {
  const newValue = [...props.modelValue]
  const index = newValue.findIndex(item => item.name === option.name)
  if (index === -1) {
    newValue.push({ ...option })
  } else {
    newValue.splice(index, 1)
  }
  emit('update:modelValue', newValue)
}

watch(() => props.field, getOptions, { immediate: true })
</script>