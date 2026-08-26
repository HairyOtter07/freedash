<template>
  <Modal :is-active="isOpen">
    <div
      class="relative flex w-full max-w-2xl flex-col gap-4 rounded-xl bg-zinc-700 p-4 text-white"
    >
      <Icon
        icon="material-symbols:close-rounded"
        class="absolute top-0 right-0 z-10 h-5 w-5 translate-x-1/2 -translate-y-1/2 rounded-full bg-white p-px text-black hover:cursor-pointer"
        @click="onCloseClick"
      />
      <slot />
    </div>
  </Modal>
</template>
<script setup>
import { Icon } from "@iconify/vue";
const isOpen = defineModel();

const onCloseClick = (event) => {
  isOpen.value = false;
};
</script>
