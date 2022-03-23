<script setup lang="ts">
import { watch, reactive, ref } from 'vue';
import { GoogleMap } from 'vue3-google-map'

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


/**
 * Using the Places API to have an autocomplete searchbox.
 * ---
 * 
 * This page is your starting point for documentation around Autocomplete.
 * https://developers.google.com/maps/documentation/javascript/places-autocomplete
 * 
 * Worth noting that for the Google Maps JS API to be used in browser 
 * you have to load it via a URL Script. This is already handled 
 * for us in this component because the package we use for our 
 * Google Map handles the loading of the URL. If you don't 
 * use it on other pages you will need to load the 
 * external script yourself.
 * 
 * First example of initialization Autocomplete
 * https://developers.google.com/maps/documentation/javascript/places-autocomplete#set-options-at-construction
 * 
 * First mention of "getting place information" from the 
 * autocomplete box. They say to use the `addListener` 
 * method on the autocomplete object.
 * https://developers.google.com/maps/documentation/javascript/places-autocomplete#get-place-information
 * 
 * A more concrete example
 * https://developers.google.com/maps/documentation/javascript/places-autocomplete#get-searchbox-information
 */

// a reference to the input element so the Places API knows where to render DOM updates
const inputRef = ref(null);

// any custom options that we may want to pass along to the autocomplete instance
const inputOptions = {}

/**
 * 
 */
watch(() => mapRef.value?.ready, (isReady) => {
  if (!isReady) return
  // initialize the autocomplete searchbox, returns an object with methods and properties
  const autocomplete = new google.maps.places.Autocomplete(inputRef.value, inputOptions);

  // listen for a selected place, and act upon it
  autocomplete.addListener('place_changed', () => {
    
    // get the selected place
    const place = autocomplete.getPlace();

    // uncomment the line below to see what the data looks like
    // console.info({place})

    /**
     * We update our userLocation, and we have a watcher above that 
     * will react to it changing and update our map for us.
     */
    userLocation.lat = place.geometry.location.lat()
    userLocation.lng = place.geometry.location.lng()
  });

}, {})

/**
 * A script to load the Google Maps JS API ourselves, 
 * instead of using some 3rd party package to do 
 * it for us. Enable if you want to use it.
 * Don't double load the script.
 */
// const googleMapsLoaded = ref(false);
// (async function loadGoogleMapsApi() {
//   try {
//    let script = document.createElement('script')
//    script.onload = () => {
//     console.info('google maps loaded')
//     googleMapsLoaded.value = true
//    }
//    script.async = true
//    script.src = `https://maps.googleapis.com/maps/api/js?key=${key}&libraries=places`
//    document.head.appendChild(script)
//   } catch (error) {
//     throw error
//   }
// })().catch((_) => {
//   console.error('yo, something went wrong')
// })

</script>

<template>
<div>
  <GoogleMap
    ref="mapRef"
    :api-key="key"
    style="width: 100%; height: 500px"
    :center="center"
    :zoom="15"
    >
    <Marker :options="{ position: center }" />
  </GoogleMap>

  <input ref="inputRef" type="text" v-model="inputValue" />
</div>
</template>
