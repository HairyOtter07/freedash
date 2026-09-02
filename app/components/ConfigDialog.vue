<template>
  <Modal :disable-backdrop="isPlacingWidget">
    <slot name="new" />
    <div v-if="!isPlacingWidget" class="w-4" />
    <div
      v-if="!isPlacingWidget"
      class="relative flex w-full max-w-2xl flex-col gap-4 rounded-xl border-2 border-(--borderColor) bg-(--widgetBackgroundColor) p-4 text-(--textColor)"
    >
      <Icon
        icon="material-symbols:close-rounded"
        class="absolute top-0 right-0 z-10 h-5 w-5 translate-x-1/2 -translate-y-1/2 rounded-full bg-(--cornerBackgroundColor) p-px text-(--cornerIconColor) hover:cursor-pointer"
        @click="onCloseClick"
      />
      <Icon
        v-if="isWidget"
        icon="material-symbols:delete-outline-rounded"
        class="absolute top-0 left-0 z-10 h-5 w-5 -translate-x-1/2 -translate-y-1/2 rounded-full bg-(--cornerBackgroundColor) p-px text-(--cornerIconColor) hover:cursor-pointer"
        @click="onDeleteClick"
      />
      <div
        v-if="isDeleteOnce"
        class="absolute top-0 left-0 flex h-5 -translate-x-2.5 -translate-y-1/2 items-center justify-center rounded-full bg-(--cornerBackgroundColor) p-2 pl-6 text-sm text-(--cornerIconColor) hover:cursor-pointer"
        @click="onDeleteClick"
      >
        delete this widget?
      </div>
      <slot />
    </div>
  </Modal>
</template>
<script setup>
import { Icon } from "@iconify/vue";
defineProps({
  isWidget: Boolean,
  isNew: Boolean,
  isPlacingWidget: Boolean,
});

const emit = defineEmits(["close", "delete"]);

const isDeleteOnce = ref(false);

const onCloseClick = (event) => {
  emit("close");
};

const onDeleteClick = (event) => {
  if (isDeleteOnce.value) {
    emit("close");
    emit("delete");
  } else {
    isDeleteOnce.value = true;
  }
};
</script>
