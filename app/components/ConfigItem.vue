<template>
  <div class="flex flex-row justify-between">
    <p>{{ name }}</p>
    <p>{{ value }}</p>
  </div>
</template>
<script setup>
defineProps({
  name: String,
  value: String,
});
</script>
