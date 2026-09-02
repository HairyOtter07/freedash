<template>
  <div
    class="fixed top-0 left-0 z-50 flex h-screen w-screen flex-row items-center justify-center"
    :class="disableBackdrop ? '' : 'bg-black/30'"
  >
    <slot />
  </div>
</template>
<script setup>
defineProps({
  disableBackdrop: Boolean,
});
</script>
