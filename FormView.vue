<template>
  <div class="md:flex h-screen bg-white dark:bg-gray-900 rounded-lg shadow-md overflow-hidden">
    <nav class="md:w-1/6 p-4 bg-gray-50 dark:bg-gray-800">
      <ul class="space-y-2">
        <li v-for="tab in tabFields" :key="tab.name">
          <button
            @click="activeTab = tab.name"
            :class="[
              'w-full text-left px-4 py-2 rounded-lg transition-colors duration-150 ease-in-out',
              activeTab === tab.name
                ? 'bg-blue-600 text-white'
                : 'text-gray-600 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700'
            ]"
          >
            {{ tab.label }}
          </button>
        </li>
      </ul>
    </nav>
    <div class=" p-6 bg-white dark:bg-gray-900">
      <div v-for="field in activeFields" :key="field.name">
        <h3 v-if="field.fieldtype === 'Section Break'" class="text-lg font-semibold text-gray-900 dark:text-white mb-4">
          {{ field.label }}
        </h3>
        <component
          v-else
          :is="getFieldComponent(field.fieldtype)"
          :field="field"
          v-model="formData[field.name]"
          class="mb-4"
        />
      </div>
    </div>
  </div>
  
</template>

<script setup>
import { ref, computed, onMounted, inject } from 'vue'
import Link from './components/Link.vue'
import Input from './components/Input.vue'
import CheckBox from './components/CheckBox.vue'
import Button from './components/Button.vue'


const props = defineProps({
  doctype: {
    type: String,
    required: true
  },
  id: {
    type: String,
    default: undefined
  },
  showFormButton: {
    type: Boolean,
    default: false
  }
})

const call = inject('$call')

const docTypeMeta = ref(null)
const activeTab = ref('')
const formData = ref({})

const tabFields = computed(() => 
  docTypeMeta.value?.fields.filter(field => field.fieldtype === 'Tab Break') || []
)

const activeFields = computed(() => {
  if (!docTypeMeta.value || !activeTab.value) return []
  
  const fields = docTypeMeta.value.fields
  const startIndex = fields.findIndex(f => f.name === activeTab.value)
  const endIndex = fields.findIndex((f, i) => i > startIndex && f.fieldtype === 'Tab Break')
  
  return fields.slice(startIndex + 1, endIndex === -1 ? undefined : endIndex)
})

const getFieldComponent = (fieldtype) => {
  switch (fieldtype) {
    case 'Link': return Link
    case 'Data': return Input
    case 'Check': return CheckBox
    case 'Button': return Button  
    default: return 'div'
  }
}

const getMeta = async () => {
  try {
    const res = await call('frappe.desk.form.load.getdoctype', {
      doctype: props.doctype,
      with_parent: 1
    })
    
    if (res?.docs?.[0]) {
      docTypeMeta.value = res.docs[0]
      activeTab.value = tabFields.value[0]?.name || ''
    } else {
      console.error('Error fetching meta data: No document found')
    }
  } catch (error) {
    console.error('Error fetching meta data:', error)
  }
}


onMounted(getMeta)
</script>