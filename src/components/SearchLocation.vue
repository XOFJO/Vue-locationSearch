<!-- This component enables users to enter a location in the search area, display it on the map, and store the search history  -->
<script setup>
import { ref, watch, watchEffect } from 'vue';
import { uid } from 'uid';
import History from './History.vue';
import Time from './Time.vue';
import AlertInfo from './AlertInfo.vue';

// Current searching location
const searchHistory = ref({
  content: '',
  invalid: null,
  errmsg: '',
});

// All past searched locations
const searchHistories = ref([
  {
    id: uid(),
    lat: 43.4722854,
    lng: -80.5448576,
    content: 'University of Waterloo',
  },
]);

// Fetch data from local storage
const fetchSearchHistories = () => {
  const savedHistory = JSON.parse(localStorage.getItem('searchHistories'));
  if (savedHistory) {
    searchHistories.value = savedHistory;
  }
};
fetchSearchHistories();

// Save searchHistories into local storage
const saveSearchHistories = () => {
  localStorage.setItem(
    'searchHistories',
    JSON.stringify(searchHistories.value)
  );
};

// A set to store selected location in the table
const deletingArray = new Set();

// Select the location
const addDelete = (historyId) => {
  deletingArray.add(historyId);
};

// Unselect the location
const removeDelete = (historyId) => {
  deletingArray.delete(historyId);
};

// Click the button to delete all selected location
const deleteAllSelected = () => {
  searchHistories.value = searchHistories.value.filter((history) => {
    return !deletingArray.has(history.id);
  });
  saveSearchHistories();
};

// Store search history into searchHistories
// Using geocoding api to convert address to latitude and longitude
const searchLocation = async () => {
  if (searchHistory.value.content !== '') {
    const apiKey = import.meta.env.VITE_GoogleMAP_API_KEY;
    try {
      const response = await fetch(
        `https://maps.googleapis.com/maps/api/geocode/json?address=${searchHistory.value.content}&key=${apiKey}`
      );
      const data = await response.json();
      searchHistories.value.push({
        id: uid(),
        lat: data.results[0].geometry.location.lat,
        lng: data.results[0].geometry.location.lng,
        content: searchHistory.value.content,
      });
      saveSearchHistories();
      searchHistory.value.content = '';
      searchHistory.value.invalid = null;
      searchHistory.value.errmsg = '';
      return;
    } catch (error) {
      searchHistory.value.invalid = true;
      searchHistory.value.errmsg = 'Invalid location name, please modify it.';
      return;
    }
  }
  searchHistory.value.invalid = true;
  searchHistory.value.errmsg = 'Cannot search empty location';
};

// When pressing the enter key, it will search the result
const handleKeyUp = (event) => {
  if (event.key === 'Enter') {
    searchLocation();
  }
};

// Map setting

let map;
let markers = [];
async function initMap() {
  // The location of Toronto
  const position = { lat: 43.7846016, lng: -79.4165248 };
  // Request needed libraries.
  //@ts-ignore
  const { Map } = await google.maps.importLibrary('maps');
  const { AdvancedMarkerElement } = await google.maps.importLibrary('marker');

  // The map, centered at Toronto
  map = new Map(document.getElementById('map'), {
    zoom: 1,
    center: position,
    mapId: 'DEMO_MAP_ID',
  });
  searchHistories.value.forEach(fetchMarkers);
}

// Fetch searched location history and set markers for each location
const fetchMarkers = async (history) => {
  const position = { lat: history.lat, lng: history.lng };
  const { AdvancedMarkerElement } = await google.maps.importLibrary('marker');
  const marker = new AdvancedMarkerElement({
    map: map,
    position: position,
    title: history.content,
  });
  markers.push(marker);
  map.setCenter(position);
  map.setZoom(6);
};
initMap();

// Update the map to show the latest searched location
watch(
  searchHistories,
  async () => {
    for (const marker of markers) {
      marker.setMap(null);
    }

    for (const history of searchHistories.value) {
      await fetchMarkers(history);
    }
    // SearchHistories.value.forEach(fetchMarkers);
    if (searchHistories.value.length === 0) {
      map.setCenter({ lat: 43.7846016, lng: -79.4165248 });
      map.setZoom(1);
    }
  },
  { deep: true }
);
</script>

<template>
  <!-- Search module -->
  <div class="flex flex-col items-center py-5">
    <div class="join">
      <input
        class="input input-bordered join-item"
        v-model="searchHistory.content"
        placeholder="Location"
        @keyup="handleKeyUp"
      />
      <button class="btn join-item rounded-r-full" @click="searchLocation">
        Search
      </button>
    </div>

    <!-- Alert  -->
    <div v-if="searchHistory.invalid" class="alert alert-error">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        class="stroke-current shrink-0 h-6 w-6"
        fill="none"
        viewBox="0 0 24 24"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z"
        />
      </svg>
      <span>{{ searchHistory.errmsg }}</span>
    </div>

    <!-- Map and Clock -->
    <div className="py-5 card lg:card-side bg-base-100 shadow-xl">
      <figure>
        <div id="map" class="h-96 w-96"></div>
      </figure>
      <div className="card-body">
        <!-- The clock and time zone -->
        <AlertInfo v-if="searchHistories.length === 0" />
        <Time
          v-for="searchHistory in searchHistories.slice(
            searchHistories.length - 1,
            searchHistories.length
          )"
          :key="searchHistory.id"
          :searchHistory="searchHistory"
        />
      </div>
    </div>

    <!-- Search history -->
    <History
      :searchHistories="searchHistories"
      @delete-select="addDelete"
      @not-delete-select="removeDelete"
      @delete-all-selected="deleteAllSelected"
    />
  </div>
</template>

<style scoped></style>
