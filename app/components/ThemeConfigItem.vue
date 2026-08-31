<template>
  <div class="flex flex-row justify-between">
    <p>{{ globalTheme[optionKey].name }}</p>
    <ThemeColorInput
      v-if="isMatchTheme"
      v-model:value="globalTheme[optionKey].value"
      v-model:match="isMatchTheme"
    />
    <ThemeColorInput
      v-else
      v-model:value="theme[optionKey].value"
      v-model:match="isMatchTheme"
    />
  </div>
</template>
<script setup>
const props = defineProps({
  optionKey: String,
});

const globalTheme = useCookie("theme");

const theme = defineModel();
const isMatchTheme = computed({
  get() {
    return !theme.value[props.optionKey];
  },
  set(newValue) {
    if (newValue) delete theme.value[props.optionKey];
    else
      theme.value[props.optionKey] = {
        name: globalTheme.value[props.optionKey].name,
        value: globalTheme.value[props.optionKey].value,
      };
  },
});
</script>
