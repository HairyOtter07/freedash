<template>
  <div class="relative h-full w-full">
    <ConfigDialog
      is-widget
      v-if="isConfigOpen"
      @close="onConfigClose"
      @delete="onDelete"
    >
      <ConfigSection title="Widget Type" v-model="options" />
      <slot />
    </ConfigDialog>
  </div>
</template>
<script setup>
defineProps({
  isConfigOpen: Boolean,
  onConfigClose: Function,
  onDelete: Function,
});

const options = defineModel();
</script>
