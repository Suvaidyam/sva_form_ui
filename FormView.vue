<template>


<div class="md:flex">
  <ul class="flex-column space-y space-y-4 text-sm font-medium text-gray-500 dark:text-gray-400 md:me-4 mb-4 md:mb-0">
    <div v-for="(tab, index) in docTypeMeta?.value?.fields" :key="index">
      <li v-if="tab?.fieldtype === 'Tab Break'">
          <a
            href="#"
            :class="['inline-flex items-center px-4 py-3 rounded-lg w-full', 
                    (activeTab == tab.name) ? 'text-white bg-blue-700 dark:bg-blue-600' : 'text-gray-500']"
            :aria-current="(activeTab == tab.name) ? 'page' : undefined"
            @click="activeTab = tab.name"
            >
            {{ tab.label }}
            
          </a>
      </li>
    </div>
  </ul>
  <div class="p-6 bg-gray-50 text-medium text-gray-500 dark:text-gray-400 dark:bg-gray-800 rounded-lg w-full">
      <div v-for="(field,index) in getFields(activeTab)" :key="index">
        <h3 v-if="field.fieldtype == 'Section Break'" class="text-lg font-bold text-gray-900 dark:text-white mb-2">{{ field.label }}</h3>
        <div v-else>
          <Link v-if="field.fieldtype == 'Link'" :field="field" />
        </div>
      </div>
  </div>
</div>

</template>
<script setup>
import { inject, onMounted, reactive, ref, watch } from 'vue'
import { Button,Input, Link, CheckBox} from './components'


const call = inject('$call')
const docTypeMeta = reactive({})
const Fields = ref([])
const props = defineProps({
  doctype: {
    type: String,
    required: true,
  },
  id: {
    type: String,
    required: false,
  },
  showFormButton: {
    type: Boolean,
    required: false,
    default: true,
  },
})
const activeTab = ref('');
const isActiveTab = (tab) => {
  return tab.name === activeTab.value
}
const getFields = (tab) => {
  let fromIndex = docTypeMeta?.value?.fields?.findIndex(f => f.name === tab);
  if(fromIndex === -1){
    return []
  }else{
    let toIndex = docTypeMeta?.value?.fields?.findIndex((f, index) => index > fromIndex && f.fieldtype === 'Tab Break');
    if(toIndex === -1){
      toIndex = docTypeMeta?.value?.fields?.length;
    }
    return docTypeMeta?.value?.fields?.slice(fromIndex + 1, toIndex);
  }
}
const getMeta = async () => {
  const res = await call('frappe.desk.form.load.getdoctype',
    { doctype: props.doctype,with_parent:1 }
  )
  if(res?.docs?.[0]){
    docTypeMeta.value = res?.docs?.[0];
    activeTab.value = docTypeMeta?.value?.fields?.find(f=> f.fieldtype == 'Tab Break')?.name;
    console.log("docTypeMeta",docTypeMeta.value,activeTab.value);
    
  }else{
    console.log('Error in fetching meta data')
  }
}
onMounted(() => {
  getMeta()
})
</script>