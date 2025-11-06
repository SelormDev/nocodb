<template>
  <div class="w-full">
    <h3 class="font-semibold">{{ table }}</h3>
    <table class="min-w-full divide-y divide-gray-200">
      <thead class="bg-gray-50">
        <tr>
          <th
            v-for="column in columns"
            :key="column"
            scope="col"
            class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
          >
            {{ column }}
          </th>
        </tr>
      </thead>
      <tbody class="bg-white divide-y divide-gray-200">
        <tr v-for="(row, index) in data" :key="index">
          <td
            v-for="column in columns"
            :key="column"
            class="px-6 py-4 whitespace-nowrap text-sm text-gray-500"
          >
            {{ row[column] }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
import { ref, watchEffect } from 'vue';
import { storeToRefs } from 'pinia';
import { useBases } from '~/store/bases';
import { useApi } from '~/composables/useApi';

const props = defineProps({
  table: {
    type: String,
    required: true,
  },
  columns: {
    type: Array,
    required: true,
  },
});

const { api } = useApi();
const { activeProjectId: baseId } = storeToRefs(useBases());
const data = ref([]);

watchEffect(async () => {
  try {
    if (props.table && props.columns.length > 0 && baseId.value) {
      const { list } = await api.dbTableRow.list('noco', baseId.value, props.table, {
        fields: props.columns,
        limit: 10,
      });
      data.value = list;
    }
  } catch (error) {
    console.error('Error fetching table data:', error);
  }
});
</script>
