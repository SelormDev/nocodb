<template>
  <div>
    <div class="mb-4">
      <label class="block text-sm font-medium text-gray-700">Table</label>
      <select
        :value="component.table"
        class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
        @change="updateTable($event.target.value)"
      >
        <option disabled value="">Please select one</option>
        <option v-for="table in tables" :key="table.id" :value="table.id">
          {{ table.title }}
        </option>
      </select>
    </div>
    <div v-if="component.table">
      <label class="block text-sm font-medium text-gray-700">Columns</label>
      <select
        multiple
        :value="component.columns"
        class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
        @change="updateColumns($event.target.value)"
      >
        <option v-for="column in columns" :key="column.id" :value="column.title">
          {{ column.title }}
        </option>
      </select>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onMounted } from 'vue';
import { storeToRefs } from 'pinia';
import { useBases } from '~/store/bases';
import { useTablesStore } from '~/store/tables';
import { useMetas } from '~/composables/useMetas';

const props = defineProps({
  component: {
    type: Object,
    required: true,
  },
});

const emit = defineEmits(['update']);

const { activeProjectId } = storeToRefs(useBases());
const tablesStore = useTablesStore();
const { activeTables: tables } = storeToRefs(tablesStore);
const { getMeta } = useMetas();

const columns = ref([]);

onMounted(async () => {
  try {
    if (activeProjectId.value) {
      await tablesStore.loadProjectTables(activeProjectId.value);
    }
  } catch (error) {
    console.error('Error loading project tables:', error);
  }
});

watch(() => props.component.table, async (newTableId) => {
  try {
    if (newTableId) {
      const meta = await getMeta(newTableId);
      columns.value = meta?.columns || [];
    }
  } catch (error) {
    console.error('Error getting table metadata:', error);
  }
});

const updateTable = (newTable) => {
  emit('update', { ...props.component, table: newTable, columns: [] });
};

const updateColumns = (newColumns) => {
  emit('update', { ...props.component, columns: newColumns });
};
</script>
