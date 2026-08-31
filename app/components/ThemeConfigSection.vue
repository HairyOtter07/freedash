<template>
  <div class="flex flex-col gap-2 p-2">
    <h1 class="text-2xl">{{ title }}</h1>
    <hr class="border-t-2 border-zinc-500" />
    <ThemeConfigItem
      v-if="props.isWidget"
      v-for="option in themeOptions"
      :option-key="option"
      v-model="theme"
    />
    <ConfigItem
      v-else
      v-for="option in Object.keys(globalTheme)"
      :name="globalTheme[option].name"
      :type="globalTheme[option].type"
      v-model="globalTheme[option].value"
    />
  </div>
</template>
<script setup>
const props = defineProps({
  title: String,
  isWidget: Boolean,
});

const globalTheme = useCookie("theme");

const theme = defineModel();
const themeOptions = [
  "widgetBackgroundColor",
  "textColor",
  "borderColor",
  "cornerBackgroundColor",
  "cornerIconColor",
];
</script>
