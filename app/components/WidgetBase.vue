<template>
  <div
    ref="widget"
    class="group relative flex h-full w-full flex-col items-center justify-center rounded-xl border-zinc-100 bg-zinc-700"
    :class="isDragging ? `z-50 cursor-grabbing shadow-xl` : ``"
  >
    <Icon
      icon="material-symbols:drag-pan-rounded"
      class="absolute top-0 left-0 h-5 w-5 -translate-1/2 rounded-full bg-white text-black"
      :class="
        isDragging
          ? `block cursor-grabbing`
          : `hidden group-hover:block hover:cursor-grab`
      "
      @mousedown="onMouseDown"
    />
    <p class="text-white">hello</p>
  </div>
</template>
<script setup>
import { Icon } from "@iconify/vue";
const emit = defineEmits(["dragStart", "dragEnd"]);
const props = defineProps({ widgetId: Number });

const widget = ref(null);
const isDragging = ref(false);
const dragStartX = ref(0);
const dragStartY = ref(0);

const onMouseDown = (event) => {
  event.preventDefault();

  dragStartX.value = event.pageX;
  dragStartY.value = event.pageY;
  isDragging.value = true;

  document.addEventListener("mousemove", onMouseMove);
  document.addEventListener("mouseup", onMouseUp);
  emit("dragStart", props.widgetId);
};

const onMouseMove = (event) => {
  const offsetX = event.pageX - dragStartX.value;
  const offsetY = event.pageY - dragStartY.value;
  widget.value.style.transform = `translate(${offsetX}px, ${offsetY}px)`;
};

const onMouseUp = (event) => {
  isDragging.value = false;
  widget.value.style.transform = "none";

  document.removeEventListener("mousemove", onMouseMove);
  document.removeEventListener("mouseup", onMouseUp);
  emit("dragEnd");
};
</script>
