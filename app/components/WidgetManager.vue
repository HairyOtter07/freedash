<template>
  <div ref="grid" class="grid h-full w-full">
    <WidgetBase
      v-for="widget in widgets"
      :key="widget.id"
      :style="`grid-column: ${widget.position.x} / span ${widget.position.width}; grid-row: ${widget.position.y} / span ${widget.position.height};`"
      :widgetId="widget.id"
      @drag-start="onDragStart"
      @drag-end="onDragEnd"
    />
    <WidgetShadow
      ref="shadow"
      class="z-40"
      :class="isDragging ? `` : `hidden`"
    />
  </div>
</template>
<script setup>
const NUM_COLS = 8;
const GAP = 32;
const PADDING = 32;

const grid = ref(null);
const shadow = ref(null);
const isDragging = ref(false);

const onDragStart = (id) => {
  const widget = widgets.find((el) => el.id == id);
  shadow.value.$el.style.gridColumn = `${widget.position.x} / span ${widget.position.width}`;
  shadow.value.$el.style.gridRow = `${widget.position.y} / span ${widget.position.height}`;
  isDragging.value = true;
};

const onDragEnd = () => {
  isDragging.value = false;
};

const debounce = (func, delay) => {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => func.apply(this, args), delay);
  };
};

const calcRowHeight = (event) => {
  if (!grid.value) return;
  grid.value.style.gridAutoRows = `${(window.innerWidth - (2 * PADDING + (NUM_COLS - 1) * GAP)) / NUM_COLS}px`;
};

const debouncedCalcHeight = debounce(calcRowHeight, 50);

onMounted(() => {
  grid.value.style.gridTemplateColumns = `repeat(${NUM_COLS}, minmax(0, 1fr))`;
  grid.value.style.gap = `${GAP}px`;
  grid.value.style.padding = `${PADDING}px`;
  calcRowHeight();
  window.addEventListener("resize", debouncedCalcHeight);
});

onUnmounted(() => {
  window.removeEventListener("resize", debouncedCalcHeight);
});

const widgets = [
  {
    id: 1,
    position: {
      x: 1,
      y: 1,
      width: 3,
      height: 3,
    },
  },
  {
    id: 2,
    position: {
      x: 4,
      y: 1,
      width: 2,
      height: 2,
    },
  },
  {
    id: 3,
    position: {
      x: 6,
      y: 2,
      width: 2,
      height: 2,
    },
  },
  {
    id: 4,
    position: {
      x: 8,
      y: 1,
      width: 1,
      height: 1,
    },
  },
];
</script>
