<template>
    <div class="flex flex-col gap-1.5">
        <h2>{{ field.label }}</h2>
        <!-- <input class="border rounded-md p-2 outline-none" :disabled="field.read_only" /> -->
        <div class="flex items-center space-x-2" v-for="(option, index) in options" :key="index">
            <input :id="option.label.toLowerCase()"
                type="checkbox"
                class="h-3 w-3 text-primary border border-[#002C77] rounded-sm"
                :checked="option.checked"
                @change="option.toggle"
            />
            <label :for="option.label.toLowerCase()" class="text-sm text-[#002C77]">{{ option.label }}</label>
        </div>
    </div>
</template>
<script setup>

import { computed, onMounted, ref, inject, watch } from 'vue';
const call = inject('$call')
const props = defineProps({
    field: {
        type: Object,
        required: false,
        default: {},
    },
})
const options = ref([])
watch(() => props.field, (value) => {
    getOptions(value)
})
const getOptions = async (value) => {
    try {
        console.log("props?.field",value,props?.field);
        
        const response = await call('frappe.client.get_list', { 
            doctype: 'Field Options',
            filters: props.field?.link_filters ? JSON.parse(props.field?.link_filters) : [],
            fields:['name','label','code']
        })
        options.value = response
        console.log("options.value",options.value?.length);
    } catch (error) {
        console.error(error)
    }
}
// onMounted(() => {
//     getOptions()
// })
</script>