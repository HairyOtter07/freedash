<template>
  <div class="flex flex-col gap-2 p-2">
    <h1 class="text-2xl">{{ title }}</h1>
    <hr class="border-t-2 border-(--borderColor)" />
    <ConfigItem
      v-for="item in Object.keys(items)"
      :name="items[item].name"
      :type="items[item].type"
      v-model="items[item].value"
    />
  </div>
</template>
<script setup>
defineProps({
  title: String,
});

const items = defineModel();
</script>
