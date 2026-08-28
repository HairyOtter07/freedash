<template>
  <div ref="grid" class="grid h-full w-full">
    <WidgetBase
      v-for="widget in widgets"
      v-slot="widgetSlotProps"
      :key="widget.id"
      :style="`grid-column: ${widget.position.x} / span ${widget.position.width}; grid-row: ${widget.position.y} / span ${widget.position.height};`"
      :widget-id="widget.id"
      @drag-start="onDragStart"
      @drag-end="onDragEnd"
    >
      <component
        :is="COMPONENT_MAP[widget.type]"
        v-model="widget.options"
        :isConfigOpen="widgetSlotProps.isConfigOpen"
        :onConfigClose="widgetSlotProps.onConfigClose"
      >
        <ConfigSection title="Widget Appearance" v-model="widget.theme" />
      </component>
    </WidgetBase>
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

const COMPONENT_MAP = {
  Countdown: resolveComponent("Countdown"),
};

const grid = ref(null);
const shadow = ref(null);

const cellSize = ref(0);
const isDragging = ref(false);
const draggingWidget = ref({});
const draggingCoords = ref({});
const cellOccupation = ref(new Map());

const calcGridCoords = (x, y, width, height, maxX, maxY = Infinity) => {
  let gridX = 0;
  if (x < PADDING) gridX = 1;
  else if (x > maxX - PADDING - width * cellSize.value - (width - 1) * GAP)
    gridX = NUM_COLS + 1 - width;
  else gridX = Math.floor((x - PADDING) / (cellSize.value + GAP)) + 1;

  let gridY = 0;
  if (y < PADDING) gridY = 1;
  else if (y > maxY - PADDING - height * cellSize.value - (height - 1) * GAP)
    gridY = NUM_COLS;
  else gridY = Math.floor((y - PADDING) / (cellSize.value + GAP)) + 1;

  return { x: gridX, y: gridY };
};

let scrollInterval = null;
const checkEdgeScroll = (event) => {
  const SCROLL_SPEED = 8;
  const INTERVAL_DELAY = 16;

  if (event.clientY < 50) {
    if (!scrollInterval) {
      scrollInterval = setInterval(() => {
        window.scrollBy(0, -SCROLL_SPEED);
      }, INTERVAL_DELAY);
    }
  } else if (window.innerHeight - event.clientY < 50) {
    if (!scrollInterval) {
      scrollInterval = setInterval(() => {
        window.scrollBy(0, SCROLL_SPEED);
      }, INTERVAL_DELAY);
    }
  } else {
    clearInterval(scrollInterval);
    scrollInterval = null;
  }
};

const getMapCell = (map, x, y) => {
  return map.get(x)?.get(y);
};

const setMapCell = (map, x, y, value) => {
  if (!map.has(x)) {
    map.set(x, new Map());
  }
  map.get(x).set(y, value);
};

const onDragStart = (id) => {
  draggingWidget.value = widgets.value.find((el) => el.id == id);
  shadow.value.$el.style.gridColumn = `${draggingWidget.value.position.x} / span ${draggingWidget.value.position.width}`;
  shadow.value.$el.style.gridRow = `${draggingWidget.value.position.y} / span ${draggingWidget.value.position.height}`;

  for (let x = 0; x < draggingWidget.value.position.width; x++) {
    for (let y = 0; y < draggingWidget.value.position.height; y++) {
      setMapCell(
        cellOccupation.value,
        draggingWidget.value.position.x + x,
        draggingWidget.value.position.y + y,
        false,
      );
    }
  }

  isDragging.value = true;
  document.addEventListener("mousemove", onDragMove);
};

const onDragMove = (event) => {
  draggingCoords.value = calcGridCoords(
    event.pageX,
    event.pageY,
    draggingWidget.value.position.width,
    draggingWidget.value.position.height,
    window.innerWidth,
  );

  checkEdgeScroll(event);

  shadow.value.$el.style.gridColumn = `${draggingCoords.value.x} / span ${draggingWidget.value.position.width}`;
  shadow.value.$el.style.gridRow = `${draggingCoords.value.y} / span ${draggingWidget.value.position.height}`;
};

const onDragEnd = () => {
  document.removeEventListener("mousemove", onDragMove);
  clearInterval(scrollInterval);
  isDragging.value = false;

  let isOccupied = false;

  for (let x = 0; x < draggingWidget.value.position.width; x++) {
    for (let y = 0; y < draggingWidget.value.position.height; y++) {
      if (
        getMapCell(
          cellOccupation.value,
          draggingCoords.value.x + x,
          draggingCoords.value.y + y,
        )
      ) {
        isOccupied = true;
        break;
      }
    }
  }

  if (!isOccupied) {
    draggingWidget.value.position.x = draggingCoords.value.x;
    draggingWidget.value.position.y = draggingCoords.value.y;
  }

  for (let x = 0; x < draggingWidget.value.position.width; x++) {
    for (let y = 0; y < draggingWidget.value.position.height; y++) {
      setMapCell(
        cellOccupation.value,
        draggingWidget.value.position.x + x,
        draggingWidget.value.position.y + y,
        true,
      );
    }
  }
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
  cellSize.value =
    (window.innerWidth - (2 * PADDING + (NUM_COLS - 1) * GAP)) / NUM_COLS;
  grid.value.style.gridAutoRows = `${cellSize.value}px`;
};

const debouncedCalcHeight = debounce(calcRowHeight, 50);

onMounted(() => {
  grid.value.style.gridTemplateColumns = `repeat(${NUM_COLS}, minmax(0, 1fr))`;
  grid.value.style.gap = `${GAP}px`;
  grid.value.style.padding = `${PADDING}px`;
  calcRowHeight();
  window.addEventListener("resize", debouncedCalcHeight);
  for (const widget of widgets.value) {
    for (let x = 0; x < widget.position.width; x++) {
      for (let y = 0; y < widget.position.height; y++) {
        setMapCell(
          cellOccupation.value,
          widget.position.x + x,
          widget.position.y + y,
          true,
        );
      }
    }
  }
});

onUnmounted(() => {
  window.removeEventListener("resize", debouncedCalcHeight);
});

const widgets = ref([
  {
    id: 1,
    type: "Countdown",
    theme: {
      backgroundColor: {
        name: "Background Color",
        type: "Text",
        value: "Gray",
      },
      textColor: {
        name: "Text Color",
        type: "Text",
        value: "White",
      },
      borderColor: {
        name: "Border Color",
        type: "Text",
        value: "None",
      },
      cornerBackgroundColor: {
        name: "Corner Button Background Color",
        type: "Text",
        value: "White",
      },
      cornerIconColor: {
        name: "Corner Button Icon Color",
        type: "Text",
        value: "Black",
      },
      size: {
        name: "Size",
        type: "Text",
        value: "2x2",
      },
    },
    options: {
      targetDate: {
        name: "Target Date",
        type: "Date",
        value: "2026-08-30T18:00",
      },
      event: {
        name: "Event",
        type: "Text",
        value: "landing in LA",
      },
    },
    position: {
      x: 1,
      y: 1,
      width: 2,
      height: 2,
    },
  },
]);
</script>
