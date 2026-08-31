<template>
  <div class="flex flex-col items-end">
    <input type="color" v-model="value" :disabled="match" />
    <div class="flex flex-row gap-2">
      <p>Match Theme</p>
      <input type="checkbox" v-model="match" />
    </div>
  </div>
</template>
<script setup>
const value = defineModel("value");
const match = defineModel("match");
</script>
