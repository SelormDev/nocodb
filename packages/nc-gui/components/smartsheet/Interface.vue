<template>
  <div class="h-full flex flex-row">
    <!-- Component Library -->
    <div class="w-64 bg-white border-r border-gray-200 p-4">
      <h2 class="text-lg font-semibold mb-4">Components</h2>
      <div class="grid grid-cols-2 gap-4">
        <div
          v-for="(component, type) in componentsRegistry"
          :key="type"
          class="p-4 border rounded-lg flex flex-col items-center justify-center cursor-pointer hover:bg-gray-100"
          draggable="true"
          @dragstart="onDragStart($event, type)"
        >
          <component :is="component.icon" class="w-8 h-8 mb-2" />
          <span>{{ type }}</span>
        </div>
      </div>
    </div>

    <!-- Canvas -->
    <div class="flex-1 bg-gray-50 p-4" @dragover.prevent @drop="onDrop">
      <grid-layout
        v-model:layout="layout"
        :col-num="12"
        :row-height="30"
        :is-draggable="true"
        :is-resizable="true"
        :vertical-compact="true"
        :use-css-transforms="true"
        @layout-updated="layoutUpdatedEvent"
      >
        <grid-item
          v-for="item in layout"
          :key="item.i"
          :x="item.x"
          :y="item.y"
          :w="item.w"
          :h="item.h"
          :i="item.i"
          @click="selectComponent(item.i)"
        >
          <component :is="componentsRegistry[item.component.type].component" v-bind="item.component" />
        </grid-item>
      </grid-layout>
    </div>

    <!-- Configuration Panel -->
    <div class="w-80 bg-white border-l border-gray-200 p-4">
      <h2 class="text-lg font-semibold mb-4">Configuration</h2>
      <div v-if="selectedComponent">
        <component
          :is="configPanelsRegistry[selectedComponent.type]"
          :component="selectedComponent"
          @update="updateComponent"
        />
      </div>
      <div v-else>
        <p class="text-gray-500">Select a component to configure.</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, shallowRef, markRaw } from 'vue';
import { GridLayout, GridItem } from 'vue3-grid-layout-next';
import TextComponent from '../interface-builder/components/TextComponent.vue';
import TextConfigPanel from '../interface-builder/config-panels/TextConfigPanel.vue';
import ButtonComponent from '../interface-builder/components/ButtonComponent.vue';
import ButtonConfigPanel from '../interface-builder/config-panels/ButtonConfigPanel.vue';
import DataTableComponent from '../interface-builder/components/DataTableComponent.vue';
import DataTableConfigPanel from '../interface-builder/config-panels/DataTableConfigPanel.vue';

// A simple icon component for the library
const TextIcon = { template: `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M19.5 14.25v-2.625a3.375 3.375 0 00-3.375-3.375h-1.5A1.125 1.125 0 0113.5 7.125v-1.5a3.375 3.375 0 00-3.375-3.375H8.25m0 12.75h7.5m-7.5 3H12M10.5 2.25H5.625c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125V11.25a9 9 0 00-9-9z" /></svg>` };
const ButtonIcon = { template: `<svg xmlns="http://www.w.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M15.042 21.672L13.684 16.6m0 0l-2.51 2.225.569-9.47 5.227 7.917-3.286-.672zM12 2.25V4.5m0 13.5v2.25m0-13.5c-1.03 0-1.9.693-2.25 1.688l-5.227 7.917h14.954l-5.227-7.917A2.25 2.25 0 0012 2.25z" /></svg>` };
const DataTableIcon = { template: `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M9 17.25v1.007a3 3 0 01-.879 2.122L7.5 21h9l-1.621-.871a3 3 0 01-.879-2.122v-1.007M9 17.25c0-.621.504-1.125 1.125-1.125h3.75c.621 0 1.125.504 1.125 1.125v0M9 17.25a2.25 2.25 0 00-2.25 2.25v1.5a2.25 2.25 0 002.25 2.25h.093c.128 0 .256.01.381.027a9.002 9.002 0 008.632 0c.125-.017.253-.027.381-.027h.093a2.25 2.25 0 002.25-2.25v-1.5a2.25 2.25 0 00-2.25-2.25v0M3 3h18M3 7.5h18M3 12h18" /></svg>` };

const componentsRegistry = {
  Text: { component: markRaw(TextComponent), icon: markRaw(TextIcon) },
  Button: { component: markRaw(ButtonComponent), icon: markRaw(ButtonIcon) },
  DataTable: { component: markRaw(DataTableComponent), icon: markRaw(DataTableIcon) },
};

const configPanelsRegistry = {
  Text: markRaw(TextConfigPanel),
  Button: markRaw(ButtonConfigPanel),
  DataTable: markRaw(DataTableConfigPanel),
};

const layout = ref([]);
const selectedComponent = ref(null);
let nextId = 0;

const onDragStart = (event, type) => {
  event.dataTransfer.setData('application/json', JSON.stringify({ type }));
};

const onDrop = (event) => {
  const { type } = JSON.parse(event.dataTransfer.getData('application/json'));
  const newItem = {
    x: (layout.value.length * 2) % 12,
    y: layout.value.length + 12, // puts item at the bottom
    w: 2,
    h: 2,
    i: nextId.toString(),
    component: {
      id: nextId.toString(),
      type,
      text: 'New ' + type,
    },
  };
  layout.value.push(newItem);
  nextId++;
};

const selectComponent = (id) => {
  const item = layout.value.find((item) => item.i === id);
  if (item) {
    selectedComponent.value = item.component;
  }
};

const updateComponent = (updatedComponent) => {
  const item = layout.value.find((item) => item.i === updatedComponent.id);
  if (item) {
    item.component = { ...item.component, ...updatedComponent };
  }
};

const layoutUpdatedEvent = (newLayout) => {
  layout.value = newLayout;
};

</script>

<style>
.vue-grid-layout {
  background: #f3f4f6;
}

.vue-grid-item:not(.vue-grid-placeholder) {
  background: #ffffff;
  border: 1px solid #e5e7eb;
}

.vue-grid-item .text {
  font-size: 24px;
  text-align: center;
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  height: 100%;
  line-height: 100%;
}

.vue-grid-item .min-max-height {
  font-size: 12px;
}

.vue-grid-item .add {
  cursor: pointer;
}
</style>
