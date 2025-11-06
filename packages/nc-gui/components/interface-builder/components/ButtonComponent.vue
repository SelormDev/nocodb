<template>
  <button :class="buttonClass">{{ text }}</button>
</template>

<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps({
  text: {
    type: String,
    default: 'Button',
  },
  buttonStyle: {
    type: String,
    default: 'primary',
  },
});

const buttonClass = computed(() => {
  return {
    'bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded': props.buttonStyle === 'primary',
    'bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded': props.buttonStyle === 'secondary',
  };
});
</script>
