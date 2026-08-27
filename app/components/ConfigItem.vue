<template>
  <div class="flex flex-row justify-between">
    <p>{{ name }}</p>
    <component :is="INPUT_COMPONENT_MAP[`${type}Input`]" v-model="value" />
  </div>
</template>
<script setup>
const props = defineProps({
  name: String,
  type: String,
});

const value = defineModel();

const INPUT_COMPONENT_MAP = {
  TextInput: resolveComponent("TextInput"),
};
</script>
