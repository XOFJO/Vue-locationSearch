<!-- show the position of the current user -->
<script setup>
import { ref } from 'vue';
// Get the position of current user
const getCurrentPos = () => {
  const success = async (pos) => {
    userPos.value.lat = pos.coords.latitude;
    userPos.value.lng = pos.coords.longitude;
    const apiKey = import.meta.env.VITE_GoogleMAP_API_KEY;
    const response = await fetch(
      `https://maps.googleapis.com/maps/api/geocode/json?latlng=${userPos.value.lat},${userPos.value.lng}&key=${apiKey}`
    );
    const data = await response.json();
    userAddress.value = data.results[0].formatted_address;
  };
  const error = (err) => {
    console.log(231);
    console.log(err);
  };
  const option = {};
  navigator.geolocation.getCurrentPosition(success, error, option);
};

const userPos = ref({ lat: '', lng: '' });
const userAddress = ref('');
</script>

<template>
  <div class="py-5">
    <button
      class="btn capitalize"
      onclick="my_modal_1.showModal()"
      @click="getCurrentPos"
    >
      Get your own location
    </button>
    <dialog id="my_modal_1" class="modal">
      <form method="dialog" class="modal-box">
        <h3 class="font-bold text-lg">Your locations is</h3>
        <p class="py-4">{{ userAddress }}</p>
        <p>Latitude: {{ userPos.lat }}, longitude: {{ userPos.lng }}</p>
        <div class="modal-action">
          <button class="btn">Close</button>
        </div>
      </form>
    </dialog>
  </div>
</template>

<style lang="scss" scoped></style>
