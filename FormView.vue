<template>
  <div class="min-h-screen bg-gray-100 dark:bg-gray-900">
    <div class="max-w-[1920px] mx-auto  bg-white dark:bg-gray-800 rounded-lg shadow-xl overflow-hidden">
      <div class="lg:flex min-h-screen">
        <nav class="lg:w-1/6 bg-gray-50 dark:bg-gray-700 p-6">
          <ul class="space-y-2">
            <li v-for="tab in tabFields" :key="tab.name">
              <button @click="setActiveTab(tab.name)" :class="[
                'w-full text-left px-4 py-2 rounded-lg transition-colors duration-150 ease-in-out',
                activeTab === tab.name
                  ? 'bg-blue-600 text-white'
                  : 'text-gray-600 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-600'
              ]">
                {{ tab.label }}
              </button>
            </li>
          </ul>
        </nav>
        <div class="lg:w-5/6 h-full overflow-y-auto">
          <form @submit.prevent="handleSubmit" class="p-6 flex flex-col h-full">
            <div class="flex-grow">
              <div v-for="field in activeFields" :key="field.name" class="mb-6">
                <h3 v-if="field.fieldtype === 'Section Break'"
                  class="text-lg font-semibold text-gray-900 dark:text-white mb-4">
                  {{ field.label }}
                </h3>
                <component v-else :is="getFieldComponent(field.fieldtype)" :field="field"
                  v-model="formData[field.name]" />
              </div>
            </div>
            <div class="mt-6 flex justify-end">
              <button v-if="!isLastTab" @click="nextTab" type="button"
                class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 mr-2">
                Next
              </button>
              <button v-if="isLastTab" type="submit"
                class="px-4 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2">
                Submit
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, inject, watch } from 'vue'
import Input from './components/Input.vue';
import Link from './components/Link.vue';
import CheckBox from './components/CheckBox.vue';
import Button from './components/Button.vue';

const props = defineProps({
  doctype: {
    type: String,
    required: true
  },
  id: {
    type: String,
    default: undefined
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

const isLastTab = computed(() => {
  const currentTabIndex = tabFields.value.findIndex(tab => tab.name === activeTab.value)
  return currentTabIndex === tabFields.value.length - 1
})

const getFieldComponent = (fieldtype) => {
  switch (fieldtype) {
    case 'Link': return Link
    case 'Data': return Input
    case 'Table MultiSelect': return CheckBox
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

const setActiveTab = (tabName) => {
  activeTab.value = tabName
}

const nextTab = () => {
  const currentIndex = tabFields.value.findIndex(tab => tab.name === activeTab.value)
  if (currentIndex < tabFields.value.length - 1) {
    activeTab.value = tabFields.value[currentIndex + 1].name
  }
}

const handleSubmit = async () => {
  console.log('Form submitted with data:', JSON.parse(JSON.stringify(formData.value)))





}

onMounted(getMeta)

watch([docTypeMeta, activeTab], () => {
  const fields = activeFields.value
  fields.forEach(field => {
    if (field.fieldtype === 'Table MultiSelect' && !formData.value[field.name]) {
      formData.value[field.name] = []
    }
  })
}, { immediate: true })
</script>
