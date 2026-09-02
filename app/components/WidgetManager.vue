<template>
  <div ref="grid" class="grid h-full w-full">
    <WidgetBase
      v-for="widget in widgets"
      v-slot="widgetSlotProps"
      :key="widget.id"
      :style="`grid-column: ${widget.position.x} / span ${widget.position.width}; grid-row: ${widget.position.y} / span ${widget.position.height};`"
      :widget-id="widget.id"
      :widget-theme="widget.theme"
      @drag-start="onDragStart"
      @drag-end="onDragEnd"
    >
      <component
        :is="WIDGET_MAP[widget.type].component"
        v-model="widget.options"
        :isConfigOpen="widgetSlotProps.isConfigOpen"
        :onConfigClose="widgetSlotProps.onConfigClose"
        :onDelete="() => deleteWidget(widget)"
      >
        <ThemeConfigSection
          title="Widget Appearance"
          is-widget
          v-model="widget.theme"
        />
      </component>
    </WidgetBase>
    <WidgetShadow
      ref="shadow"
      class="z-40"
      :class="isDragging ? `` : `hidden`"
    />
    <div class="fixed right-3 bottom-3 z-40 flex flex-row gap-3">
      <Icon
        icon="material-symbols:settings-rounded"
        class="h-8 w-8 rounded-full bg-(--cornerBackgroundColor) p-px text-(--cornerIconColor) shadow-xl hover:cursor-pointer"
        @click="onThemeConfigClick"
      />
      <Icon
        icon="material-symbols:add-rounded"
        class="h-8 w-8 rounded-full bg-(--cornerBackgroundColor) p-px text-(--cornerIconColor) shadow-xl hover:cursor-pointer"
        @click="onAddClick"
      />
    </div>
    <ConfigDialog v-if="isThemeConfigOpen" @close="onThemeConfigClose">
      <ThemeConfigSection title="Theme" v-model="theme" />
    </ConfigDialog>
    <WidgetSelector
      v-if="isWidgetSelectorOpen"
      :widget-map="WIDGET_MAP"
      @select="onWidgetSelectorSelect"
      @close="onWidgetSelectorClose"
    />
    <ConfigDialog
      v-if="isNewWidgetSelected"
      :is-placing-widget="isDragging"
      is-new
      @close="onNewWidgetConfigClose"
    >
      <template #new>
        <WidgetBase
          ref="newWidget"
          :widget-id="addingWidget.id"
          :widget-theme="addingWidget.theme"
          is-new-widget
          :style="`width: ${addingWidget.position.width * cellSize + (addingWidget.position.width - 1) * GAP}px; height: ${addingWidget.position.height * cellSize + (addingWidget.position.height - 1) * GAP}px`"
          @drag-start="onNewWidgetDragStart"
          @drag-end="onNewWidgetDragEnd"
        >
          <component
            :is="WIDGET_MAP[addingWidget.type].component"
            v-model="addingWidget.options"
          />
        </WidgetBase>
      </template>
      <ConfigSection
        :title="WIDGET_MAP[addingWidget.type].name"
        v-model="addingWidget.options"
      />
      <div class="flex w-full items-center justify-center">
        When you're ready, drag the widget onto the dashboard or click the X to
        cancel.
      </div>
    </ConfigDialog>
  </div>
</template>
<script setup>
import { Icon } from "@iconify/vue";

const NUM_COLS = 8;
const GAP = 32;
const PADDING = 32;

const WIDGET_MAP = {
  Countdown: {
    name: "Countdown",
    description: "Displays a countdown to an event.",
    component: resolveComponent("Countdown"),
    options: {
      targetDate: {
        name: "Target Date",
        type: "Date",
      },
      event: {
        name: "Event",
        type: "Text",
      },
    },
    sizes: [[2, 2]],
  },
};

const DEFAULT_VALUES = {
  Text: () => "",
  Date: () => {
    var now = new Date();
    now.setMinutes(now.getMinutes() - now.getTimezoneOffset());
    return now.toISOString().slice(0, 16);
  },
  Color: () => "#000000",
};

const grid = ref(null);
const shadow = ref(null);
const newWidget = ref(null);

const widgets = ref([]);
const cellSize = ref(0);
const isDragging = ref(false);
const draggingWidget = ref({});
const draggingCoords = ref({});
const cellOccupation = ref(new Map());
const isThemeConfigOpen = ref(false);
const isWidgetSelectorOpen = ref(false);
const isNewWidgetSelected = ref(false);
const addingWidget = ref({});

const onThemeConfigClick = () => {
  isThemeConfigOpen.value = true;
};
const onThemeConfigClose = () => {
  isThemeConfigOpen.value = false;
};

const onAddClick = () => {
  isWidgetSelectorOpen.value = true;
};

const onWidgetSelectorSelect = (selected) => {
  addingWidget.value = {
    id: widgets.value.at(-1).id + 1,
    type: selected,
    theme: {},
    options: WIDGET_MAP[selected].options,
    position: {
      width: WIDGET_MAP[selected].sizes[0][0],
      height: WIDGET_MAP[selected].sizes[0][1],
    },
  };
  for (const option of Object.keys(WIDGET_MAP[selected].options)) {
    addingWidget.value.options[option].value =
      DEFAULT_VALUES[WIDGET_MAP[selected].options[option].type]();
  }
  isNewWidgetSelected.value = true;
};

const onWidgetSelectorClose = () => {
  isWidgetSelectorOpen.value = false;
};

const onNewWidgetConfigClose = () => {
  isNewWidgetSelected.value = false;
};

const onNewWidgetDragStart = () => {
  const rect = newWidget.value.$el.getBoundingClientRect();
  newWidget.value.$el.style.position = "absolute";
  newWidget.value.$el.style.top = `${rect.top}px`;
  newWidget.value.$el.style.left = `${rect.left}px`;

  const coords = calcGridCoords(
    rect.x,
    rect.y,
    addingWidget.value.position.width,
    addingWidget.value.position.height,
    window.innerWidth,
  );
  addingWidget.value.position.x = coords.x;
  addingWidget.value.position.y = coords.y;
  draggingWidget.value = JSON.parse(JSON.stringify(addingWidget.value));
  shadow.value.$el.style.gridColumn = `${draggingWidget.value.position.x} / span ${draggingWidget.value.position.width}`;
  shadow.value.$el.style.gridRow = `${draggingWidget.value.position.y} / span ${draggingWidget.value.position.height}`;

  isDragging.value = true;
  document.addEventListener("mousemove", onDragMove);
};

const onNewWidgetDragEnd = () => {
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
    widgets.value.push(draggingWidget.value);
  }

  onNewWidgetConfigClose();
};

const deleteWidget = (widget) => {
  for (let x = 0; x < widget.position.width; x++) {
    for (let y = 0; y < widget.position.height; y++) {
      setMapCell(
        cellOccupation.value,
        widget.position.x + x,
        widget.position.y + y,
        false,
      );
    }
  }
  const index = widgets.value.indexOf(widget);
  widgets.value.splice(index, 1);
};

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
  if (!localStorage.getItem("widgets")) {
    localStorage.setItem("widgets", "[]");
  }
  widgets.value = JSON.parse(localStorage.getItem("widgets"));

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

  watchEffect(() => {
    if (!process.client) return;
    localStorage.setItem("widgets", JSON.stringify(widgets.value));
  });
});

onUnmounted(() => {
  window.removeEventListener("resize", debouncedCalcHeight);
});

const theme = useCookie("theme");

theme.value ||= {
  backgroundColor: {
    name: "Background Color",
    type: "Color",
    value: "#18181b",
  },
  widgetBackgroundColor: {
    name: "Widget Background Color",
    type: "Color",
    value: "#3f3f46",
  },
  textColor: {
    name: "Text Color",
    type: "Color",
    value: "#ffffff",
  },
  borderColor: {
    name: "Border Color",
    type: "Color",
    value: "#3f3f46",
  },
  cornerBackgroundColor: {
    name: "Corner Button Background Color",
    type: "Color",
    value: "#ffffff",
  },
  cornerIconColor: {
    name: "Corner Button Icon Color",
    type: "Color",
    value: "#000000",
  },
};

watchEffect(() => {
  if (!document) return;
  for (const key of Object.keys(theme.value)) {
    document.documentElement.style.setProperty(
      `--${key}`,
      theme.value[key].value,
    );
  }
});
</script>
