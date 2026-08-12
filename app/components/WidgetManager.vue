<template>
  <div ref="grid" class="grid h-full w-full">
    <WidgetBase
      v-for="widget in widgets"
      :style="`grid-column: ${widget.x} / span ${widget.width}; grid-row: ${widget.y} / span ${widget.height};`"
    />
  </div>
</template>
<script setup>
const NUM_COLS = 8;
const GAP = 32;
const PADDING = 32;
const grid = ref(null);

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
    x: 1,
    y: 1,
    width: 2,
    height: 2,
  },
  {
    x: 3,
    y: 1,
    width: 1,
    height: 1,
  },
  {
    x: 4,
    y: 1,
    width: 3,
    height: 3,
  },
  {
    x: 7,
    y: 1,
    width: 2,
    height: 2,
  },
  {
    x: 3,
    y: 2,
    width: 1,
    height: 1,
  },
  {
    x: 1,
    y: 3,
    width: 3,
    height: 3,
  },
  {
    x: 4,
    y: 4,
    width: 1,
    height: 1,
  },
  {
    x: 4,
    y: 5,
    width: 1,
    height: 1,
  },
  {
    x: 5,
    y: 4,
    width: 2,
    height: 2,
  },
  {
    x: 7,
    y: 3,
    width: 1,
    height: 1,
  },
  {
    x: 8,
    y: 3,
    width: 1,
    height: 1,
  },
  {
    x: 7,
    y: 4,
    width: 2,
    height: 2,
  },
];
</script>
