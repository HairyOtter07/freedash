<template>
  <Modal>
    <div
      class="relative flex w-full max-w-2xl flex-col gap-4 rounded-xl border-2 border-(--borderColor) bg-(--widgetBackgroundColor) p-4 text-(--textColor)"
    >
      <Icon
        icon="material-symbols:close-rounded"
        class="absolute top-0 right-0 z-10 h-5 w-5 translate-x-1/2 -translate-y-1/2 rounded-full bg-(--cornerBackgroundColor) p-px text-(--cornerIconColor) hover:cursor-pointer"
        @click="onCloseClick"
      />
      <slot />
    </div>
  </Modal>
</template>
<script setup>
import { Icon } from "@iconify/vue";
const emit = defineEmits(["close"]);

const onCloseClick = (event) => {
  emit("close");
};
</script>
