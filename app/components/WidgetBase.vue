<template>
  <div
    ref="widget"
    class="group relative flex h-full w-full flex-col items-center justify-center rounded-xl border border-(--borderColor) bg-(--widgetBackgroundColor) text-(--textColor)"
    :class="isDragging ? `z-50 cursor-grabbing shadow-xl` : ``"
  >
    <Icon
      icon="material-symbols:drag-pan-rounded"
      class="absolute top-0 left-0 z-10 h-5 w-5 -translate-1/2 rounded-full bg-(--cornerBackgroundColor) p-px text-(--cornerIconColor)"
      :class="
        isDragging
          ? `block cursor-grabbing`
          : `hidden group-hover:block hover:cursor-grab`
      "
      @mousedown="onDragStart"
    />
    <Icon
      icon="material-symbols:settings-rounded"
      class="absolute top-0 right-0 z-10 hidden h-5 w-5 translate-x-1/2 -translate-y-1/2 rounded-full bg-(--cornerBackgroundColor) p-px text-(--cornerIconColor) group-hover:block hover:cursor-pointer"
      @click="onConfigClick"
    />
    <slot :isConfigOpen="isConfigOpen" :onConfigClose="onConfigClose" />
  </div>
</template>
<script setup>
import { Icon } from "@iconify/vue";
const emit = defineEmits(["dragStart", "dragEnd"]);
const props = defineProps({ widgetId: Number, widgetTheme: Object });

const widget = ref(null);
const isDragging = ref(false);
const dragStartX = ref(0);
const dragStartY = ref(0);
const isConfigOpen = ref(false);

const onConfigClick = (event) => {
  isConfigOpen.value = true;
};

const onConfigClose = () => {
  isConfigOpen.value = false;
};

const onDragStart = (event) => {
  event.preventDefault();

  dragStartX.value = event.pageX;
  dragStartY.value = event.pageY;
  isDragging.value = true;

  document.addEventListener("mousemove", onDragMove);
  document.addEventListener("mouseup", onDragEnd);
  emit("dragStart", props.widgetId);
};

const onDragMove = (event) => {
  const offsetX = event.pageX - dragStartX.value;
  const offsetY = event.pageY - dragStartY.value;
  widget.value.style.transform = `translate(${offsetX}px, ${offsetY}px)`;
};

const onDragEnd = (event) => {
  isDragging.value = false;
  widget.value.style.transform = "none";

  document.removeEventListener("mousemove", onDragMove);
  document.removeEventListener("mouseup", onDragEnd);
  emit("dragEnd");
};

const themeOptions = [
  "widgetBackgroundColor",
  "textColor",
  "borderColor",
  "cornerBackgroundColor",
  "cornerIconColor",
];
watchEffect(() => {
  if (!widget.value || !props.widgetTheme) return;
  for (const option of themeOptions) {
    if (props.widgetTheme[option])
      widget.value.style.setProperty(
        `--${option}`,
        props.widgetTheme[option].value,
      );
    else widget.value.style.removeProperty(`--${option}`);
  }
});
</script>
