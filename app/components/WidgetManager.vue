<template>
  <div ref="grid" class="grid h-full w-full auto-rows-43.75 grid-cols-8">
    <WidgetBase
      v-for="widget in widgets"
      :style="`grid-column: ${widget.x} / span ${widget.width}; grid-row: ${widget.y} / span ${widget.height};`"
    />
  </div>
</template>
<script setup>
const grid = ref(null);
const calcRowHeight = (event) => {
  grid.value.style = `grid-auto-rows: ${window.innerWidth / 8}px;`;
};

onMounted(() => {
  calcRowHeight();
  window.addEventListener("resize", calcRowHeight);
});

onUnmounted(() => {
  window.removeEventListener("resize", calcRowHeight);
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
