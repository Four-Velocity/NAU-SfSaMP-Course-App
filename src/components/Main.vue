<template class="main">
  <v-toolbar color="primary">
    <v-toolbar-title> Курсова робота</v-toolbar-title>
  </v-toolbar>

  <v-container fluid>
    <v-row align="center">
      <v-col cols="4">
        <v-subheader> Поточне місто </v-subheader>
      </v-col>
      <v-col cols="8">
        <v-select
          v-model="select"
          :items="cities"
          label="Оберіть місто"
          prepend-icon="mdi-map"
          return-object
          single-line
        ></v-select>
      </v-col>
    </v-row>
  </v-container>

  <GoogleMap
    :api-key="key"
    style="width: 100%; height: 400px"
    :center="select.center"
    :zoom="select.zoom"
    :disable-default-ui="true"
    @click="defineLocation"
  >
    <template v-if="showMarkers">
        <Marker :options="userMarker" />
    </template>
  </GoogleMap>
</template>

<script>
import { GoogleMap, Marker } from "vue3-google-map";
import citiesArray from "@/res/cities.json";
import { faMapPin, faStreetView } from "@fortawesome/free-solid-svg-icons";

citiesArray.sort((a, b) => a.title.localeCompare(b.title));
// debugger
export default {
  components: { GoogleMap, Marker },
  data: () => ({
    key: import.meta.env.VITE_GMAPS_KEY,
    showMarkers: true,
    select: {
      title: "Київ",
      value: "KV",
      center: { lat: 50.448287, lng: 30.522876 },
      zoom: 11,
    },
    cities: citiesArray,
    userMarker: {
      icon: {
        path: faStreetView.icon[4],
        scale: 0.1,
        anchor: {
            x: faStreetView.icon[0] / 2,
            y: faStreetView.icon[1]
        },
        fillColor: "#388E3C",
        fillOpacity: 1,
        strokeWeight: 2,
        strokeColor: "#000000",
      },
    },
  }),
  methods: {
    defineLocation(event) {
        this.showMarkers = true;
        const latLng = {lat: event.latLng.lat(), lng: event.latLng.lng()};
        let userMarkerCopy = {...this.userMarker};
        userMarkerCopy.position = latLng;
        this.userMarker = userMarkerCopy;
    }
  },
  watch: {
    select(newSelect, oldSelect) {
        if (newSelect.value !== oldSelect.value) {
            this.showMarkers = false;
        }
    }
  }
};
</script>
