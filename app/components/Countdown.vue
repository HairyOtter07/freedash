<template>
  <div class="relative h-full w-full">
    <div
      class="absolute top-1/2 left-1/2 flex -translate-1/2 flex-col items-center justify-center"
    >
      <h1 class="text-7xl font-bold">{{ isComplete ? "🎉" : num }}</h1>
      <h2 v-if="!isComplete" class="text-lg">{{ unit }}</h2>
    </div>
    <p class="absolute inset-x-4 bottom-2 text-center text-lg">
      {{ isComplete ? "" : "until " }}<span class="font-bold">{{ event }}</span>
    </p>
    <ConfigDialog v-if="isConfigOpen" @close="onConfigClose">
      <ConfigSection title="Countdown" v-model="options" />
      <slot />
    </ConfigDialog>
  </div>
</template>
<script setup>
import dayjs from "dayjs";
defineProps({
  isConfigOpen: Boolean,
  onConfigClose: Function,
});

const options = defineModel();

const targetDate = computed(() => options.value.targetDate.value);
const event = computed(() => options.value.event.value);

const num = ref(0);
const unit = ref("");
const isComplete = ref(false);

const updateDiff = () => {
  const target = dayjs(targetDate.value);
  const now = dayjs();

  if (target.diff(now) < 1000) {
    num.value = 0;
    unit.value = "seconds";
    isComplete.value = true;
    clearInterval(updateInterval);
    return;
  }

  if (target.diff(now, "day") > 0) {
    num.value = target.diff(now, "day");
    unit.value = num.value > 1 ? "days" : "day";
  } else if (target.diff(now, "hour") > 0) {
    num.value = target.diff(now, "hour");
    unit.value = num.value > 1 ? "hours" : "hour";
  } else if (target.diff(now, "minute") > 0) {
    num.value = target.diff(now, "minute");
    unit.value = num.value > 1 ? "minutes" : "minute";
  } else {
    num.value = target.diff(now, "second");
    unit.value = num.value > 1 ? "seconds" : "second";
  }
};

let updateInterval = null;
onMounted(() => {
  updateDiff();
  updateInterval = setInterval(updateDiff, 1000);
});

onUnmounted(() => {
  clearInterval(updateInterval);
});

watch(options, () => {
  isComplete.value = false;
  clearInterval(updateInterval);
  updateDiff();
  updateInterval = setInterval(updateDiff, 1000);
});
</script>
