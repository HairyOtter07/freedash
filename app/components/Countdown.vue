<template>
  <div class="relative h-full w-full text-white">
    <div
      class="absolute top-1/2 left-1/2 flex -translate-1/2 flex-col items-center justify-center"
    >
      <h1 class="text-7xl font-bold">{{ isComplete ? "🎉" : num }}</h1>
      <h2 v-if="!isComplete" class="text-lg">{{ unit }}</h2>
    </div>
    <p class="absolute inset-x-4 bottom-2 text-center text-lg">
      {{ isComplete ? "" : "until "
      }}<span class="font-bold">{{ props.event }}</span>
    </p>
  </div>
</template>
<script setup>
import dayjs from "dayjs";

const props = defineProps({
  targetDate: Date,
  event: String,
});

const num = ref(0);
const unit = ref("");
const isComplete = ref(false);

const updateDiff = () => {
  const targetDate = dayjs(props.targetDate);
  const now = dayjs();

  if (targetDate.diff(now) < 1000) {
    num.value = 0;
    unit.value = "seconds";
    isComplete.value = true;
    clearInterval(updateInterval);
    return;
  }

  if (targetDate.diff(now, "day") > 0) {
    num.value = targetDate.diff(now, "day");
    unit.value = num.value > 1 ? "days" : "day";
  } else if (targetDate.diff(now, "hour") > 0) {
    num.value = targetDate.diff(now, "hour");
    unit.value = num.value > 1 ? "hours" : "hour";
  } else if (targetDate.diff(now, "minute") > 0) {
    num.value = targetDate.diff(now, "minute");
    unit.value = num.value > 1 ? "minutes" : "minute";
  } else {
    num.value = targetDate.diff(now, "second");
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

watch(props, () => {
  isComplete.value = false;
  clearInterval(updateInterval);
  updateDiff();
  updateInterval = setInterval(updateDiff, 1000);
});
</script>
