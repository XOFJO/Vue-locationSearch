<!-- This is the table component to store search locations -->
<script setup>
import HistoryItem from './HistoryItem.vue';
import { defineEmits, ref, watch } from 'vue';

defineEmits(['delete-all-selected', 'not-delete-select', 'delete-select']);
const startPage = ref(0);
const prevPage = () => {
  startPage.value -= 1;
};
const nextPage = () => {
  startPage.value += 1;
};
const props = defineProps({
  searchHistories: {
    type: Array,
    required: true,
  },
});

watch(props.searchHistories, (newValue) => {
  startPage.value = Math.floor(newValue.length / 11);
});

// SearchHistories prop from SearchLocation component, it contains array of object containing search location information
</script>

<template>
  <div class="py-5 overflow-x-auto">
    <table class="table">
      <!-- head -->
      <thead>
        <tr>
          <th>
            <button
              class="btn btn-sm btn-primary"
              @click="$emit('delete-all-selected')"
            >
              Delete
            </button>
          </th>
          <th>Latitude</th>
          <th>Longitude</th>
          <th>Location</th>
        </tr>
      </thead>
      <tbody>
        <!-- Rows of location information with checkbox -->
        <HistoryItem
          v-for="searchHistory in searchHistories.slice(
            10 * startPage,
            10 * startPage + 10
          )"
          :key="searchHistory.id"
          :searchHistory="searchHistory"
          @not-delete-select="$emit('not-delete-select', searchHistory.id)"
          @delete-select="$emit('delete-select', searchHistory.id)"
        />
      </tbody>
    </table>
  </div>

  <!-- pagination -->
  <div class="join py-5 m-auto">
    <button class="join-item btn" :disabled="startPage === 0" @click="prevPage">
      «
    </button>
    <button class="join-item btn">Page {{ startPage + 1 }}</button>
    <button
      class="join-item btn"
      :disabled="startPage === Math.floor(searchHistories.length / 11)"
      @click="nextPage"
    >
      »
    </button>
  </div>
</template>

<style lang="scss" scoped></style>
