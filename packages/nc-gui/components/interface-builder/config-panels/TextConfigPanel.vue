<template>
  <div>
    <label class="block text-sm font-medium text-gray-700">Text</label>
    <input
      type="text"
      :value="component.text"
      class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
      @input="updateText($event.target.value)"
    />
  </div>
</template>

<script setup lang="ts">
const props = defineProps({
  component: {
    type: Object,
    required: true,
  },
});

const emit = defineEmits(['update']);

const updateText = (newText) => {
  emit('update', { ...props.component, text: newText });
};
</script>
