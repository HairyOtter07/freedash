<template>
  <Modal>
    <div
      class="relative flex w-full max-w-2xl flex-col gap-4 rounded-xl border-2 border-(--borderColor) bg-(--widgetBackgroundColor) p-4 text-(--textColor)"
    >
      <h1 class="text-2xl">Add Widget</h1>
      <hr class="border-t-2 border-(--borderColor)" />
      <Icon
        icon="material-symbols:close-rounded"
        class="absolute top-0 right-0 z-10 h-5 w-5 translate-x-1/2 -translate-y-1/2 rounded-full bg-(--cornerBackgroundColor) p-px text-(--cornerIconColor) hover:cursor-pointer"
        @click="onCloseClick"
      />
      <div class="flex flex-row justify-between">
        <p>Widget Type</p>
        <DropdownInput v-model="value" />
      </div>
      <div v-if="value.selected" class="flex flex-row justify-between">
        <p>{{ widgetMap[value.selected].description }}</p>
        <button
          class="rounded-lg border border-(--borderColor) px-2 py-1 hover:cursor-pointer hover:bg-black/10"
          @click="onConfirmClick"
        >
          Confirm
        </button>
      </div>
    </div>
  </Modal>
</template>
<script setup>
import { Icon } from "@iconify/vue";

const props = defineProps({
  widgetMap: Object,
});

const emit = defineEmits(["close", "select"]);

const onCloseClick = (event) => {
  emit("close");
};

const onConfirmClick = (event) => {
  emit("close");
  emit("select", value.value.selected);
};

const value = ref({ selected: "", options: [] });
watchEffect(() => {
  if (!props.widgetMap) return;
  const options = [];
  for (const key of Object.keys(props.widgetMap)) {
    options.push({ key: key, name: props.widgetMap[key].name });
  }
  value.value.options = options;
});
</script>
