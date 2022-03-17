<script setup lang="ts">
// imports
import { reactive, ref } from '@vue/reactivity'
import { watch } from 'vue';
import { GoogleMap, Marker } from 'vue3-google-map'

// reference to the map element
const mapRef = ref(null);

// starting position for map
const center = { lat: 40.689247, lng: -74.044502 }

// api key
const key = import.meta.env.VITE_MAPS_API_KEY

// place to store user lat/lng
const userLocation = reactive({
  lat: 0,
  lng: 0
})

// when this component runs attempt to get the User's location
// provide a callback to handle the provided position
navigator.geolocation.getCurrentPosition((pos) => {
  // update the User's location
  userLocation.lat = pos.coords.latitude
  userLocation.lng = pos.coords.longitude
})

// watch for changes to the User's Location
watch([userLocation, () => mapRef.value?.ready], ([loc, isReady]) => {
  if (isReady && loc.lat && loc.lng) {
    // if the map is ready and the userLocation has been set/changed, 
    // let's use the component's internal method for moving the map
    mapRef.value?.map.panTo(loc)
  }
}, {})

</script>

<template>
  <GoogleMap
    ref="mapRef"
    :api-key="key"
    style="width: 100%; height: 500px"
    :center="center"
    :zoom="15"
    >
    <Marker :options="{ position: center }" />
  </GoogleMap>
</template>
