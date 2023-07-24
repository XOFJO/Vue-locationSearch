<script setup>
import { defineEmits } from 'vue';
import { uid } from 'uid';
const emit = defineEmits(['delete-select', 'not-delete-select']);

// searchHistory is an object contains location information and unique ID
const props = defineProps(
  {
    searchHistory: {
      type: Object,
      required: true,
    },
  },
  { deep: true }
);

//  Emit an event to add the current searchHistory to the delete array of the event or remove it from the delete array of the event
const handleCheck = (e) => {
  if (e.target.checked) {
    emit('delete-select', props.searchHistory.id);
  } else {
    emit('not-delete-select', props.searchHistory.id);
  }
};
</script>

<template>
  <tr>
    <th>
      <input type="checkbox" class="checkbox" @click="handleCheck" />
    </th>
    <td>{{ searchHistory.lat }}</td>
    <td>{{ searchHistory.lng }}</td>
    <td>{{ searchHistory.content }}</td>
  </tr>
</template>

<style lang="scss" scoped></style>
