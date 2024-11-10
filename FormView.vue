<template>
  <Button /> 
  <div class="p-2">
  <Input />
  <CheckBox :checked="false" label="Funder Diagnostic"/>
</div>
</template>
<script setup>
import { inject, onMounted, ref, watch } from 'vue'
import { Button,Input ,CheckBox} from './components'


const call = inject('$call')
const fields = ref([])
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
const getMeta = async () => {
  const res = await call('pwit.controllers.api.get_meta',
    { doctype: props.doctype }
  )
  fields.value = res.fields
}
onMounted(() => {
  getMeta()
})
</script>