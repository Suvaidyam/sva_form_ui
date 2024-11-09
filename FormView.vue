<template>
    message:    {{state.message}}   
    <Button></Button>
</template>
<script setup>
  import { reactive, ref } from 'vue'
  import {Button} from './components/index'
  const state = reactive({
      message:"Hello World",
  })
</script>