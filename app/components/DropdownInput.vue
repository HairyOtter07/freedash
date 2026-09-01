<template>
  <select v-model="value.selected">
    <option disabled value="">Please select one</option>
    <option v-for="option of value.options" :value="option.key">
      {{ option.name }}
    </option>
  </select>
</template>
<script setup>
const value = defineModel();
</script>
