<template>
  <div class="min-h-screen bg-gray-100 dark:bg-gray-900">
    <div class="max-w-[1920px] min-h-screen mx-auto bg-white dark:bg-gray-800 rounded-lg shadow-xl overflow-hidden">
      <div :class="{ 'lg:flex min-h-screen': props.sidebar && !props.section }">
        <!-- Sidebar / Top Navigation -->
        <nav v-if="!props.section" :class="[
          props.sidebar ? 'lg:w-1/6 bg-gray-50 dark:bg-gray-700 p-6' : 'w-full bg-gray-50 dark:bg-gray-700 p-4'
        ]">
          <ul :class="[
            props.sidebar ? 'space-y-2' : 'flex space-x-2 overflow-x-auto'
          ]">
            <li v-for="tab in tabFields" :key="tab.name" :class="{ 'flex-shrink-0': !props.sidebar }">
              <button @click="setActiveTab(tab.name)" :class="[
                'text-left px-4 py-2 rounded-lg transition-colors duration-150 ease-in-out',
                activeTab === tab.name
                  ? 'bg-secondary text-white'
                  : 'text-gray-600 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-600',
                props.sidebar ? 'w-full' : 'whitespace-nowrap'
              ]">
                {{ tab.label }}
              </button>
            </li>
          </ul>
        </nav>
        <!-- Main Content -->
        <div :class="[props.sidebar && !props.section ? 'lg:w-5/6' : 'w-full', 'h-full overflow-y-auto']">
          <form @submit.prevent="handleSubmit" class="p-6 flex flex-col h-full">
            <div class="flex-grow">
              <template v-if="props.section">
                <div v-for="(section, index) in allSections" :key="index" class="mb-6">
                  
                  <div @click="toggleSection(index)" class="flex items-center justify-between cursor-pointer bg-gray-100 dark:bg-gray-700 p-4 rounded-lg mb-2">
                    <h3 class="text-lg font-semibold text-gray-900 dark:text-white">
                      {{ section.label }}
                    </h3>
                    <ChevronDownIcon :class="['w-5 h-5 transition-transform', { 'transform rotate-180': openSections[index] }]" />
                  </div>
                  <div v-show="openSections[index]" class="pl-4">
                    <div v-for="field in section.fields" :key="field.name" class="mb-4">
                      <component :is="getFieldComponent(field.fieldtype)" :field="field"
                        v-model="formData[field.fieldname]" />
                    </div>
                  </div>
                </div>
              </template>
              <template v-else>
                <div v-for="field in activeFields" :key="field.name" class="mb-6">
                  <h3 v-if="field.fieldtype === 'Section Break'"
                    class="text-lg font-semibold text-gray-900 dark:text-white mb-4">
                    {{ field.label }}
                  </h3>
                  <component v-else :is="getFieldComponent(field.fieldtype)" :field="field"
                    v-model="formData[field.fieldname]" />
                </div>
              </template>
            </div>
            <div class="mt-6 flex justify-end">
              <button v-if="!props.section && !isLastTab" @click="nextTab" type="button"
                class="px-4 py-2 bg-secondary  text-white rounded-md focus:ring-offset-2 mr-2">
                Next
              </button>
              <button v-if="props.section || isLastTab" type="submit"
                class="px-4 py-2 bg-secondary text-white rounded-md hover:bg-primary focus:outline-none">
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
import { ChevronDownIcon } from 'lucide-vue-next'
import Input from './components/Input.vue'
import Link from './components/Link.vue'
import CheckBox from './components/CheckBox.vue'
import Button from './components/Button.vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const props = defineProps({
  doctype: {
    type: String,
    required: true
  },
  sidebar: {
    type: Boolean,
    default: false,
    required: true
  },
  isRoute: {
    type: String,
    default: '',
    required: false
  },
  section: {
    type: Boolean,
    default: false ,
    required: true
  }
})

const call = inject('$call')

const docTypeMeta = ref(null)
const activeTab = ref('')
const formData = ref({})
const openSections = ref([])

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

const allSections = computed(() => {
  if (!docTypeMeta.value) return []

  const sections = []
  let currentSection = null

  docTypeMeta.value.fields.forEach(field => {
    if (field.fieldtype === 'Tab Break') {
      // Skip Tab Break fields when section is true
      return
    }
    if (field.fieldtype === 'Section Break') {
      if (currentSection) {
        sections.push(currentSection)
      }
      currentSection = { label: field.label,collapsible:field.collapsible, fields: [] }
    } else if (currentSection) {
      currentSection.fields.push(field)
    }
  })

  if (currentSection) {
    sections.push(currentSection)
  }

  return sections
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
      openSections.value = new Array(allSections.value.length).fill(true)
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

const toggleSection = (index) => {
  openSections.value[index] = !openSections.value[index]
}

const handleSubmit = async () => {
  console.log('Form submitted with data:', JSON.parse(JSON.stringify(formData.value)))
  try {
    const res = await call('frappe.desk.form.save.savedocs', {
      doc: JSON.stringify({
        doctype: props.doctype,
        ...formData.value
      }),
      action: 'Save'
    })
    console.log('Saved:', res)
    if(res && props.isRoute) {
      router.push(props.isRoute)
    }
  } catch (err) {
    console.error('Error saving form:', err)
  }
}

onMounted(getMeta)

watch([docTypeMeta, activeTab], () => {
  const fields = props.section ? docTypeMeta.value?.fields : activeFields.value
  fields?.forEach(field => {
    if (field.fieldtype === 'Table MultiSelect' && !formData.value[field.fieldname]) {
      formData.value[field.fieldname] = []
    }
  })
}, { immediate: true })
</script>