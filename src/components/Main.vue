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
    id="map"
    @click="defineLocation"
  >
    <template v-if="showMarkers">
      <Marker :options="userMarker" />
      <template v-for="hotel in hotels" v-bind:key="hotel.id">
        <Marker :options="hotel" />
      </template>
    </template>
  </GoogleMap>
  <template v-if="closest">
    <v-card class="mx-auto">
      <v-img :src="closest.imgUrl" height="200px" cover></v-img>
      <v-card-title>
        {{ closest.name }}
      </v-card-title>
      <v-card-subtitle>
        {{ closest.distance.toFixed(0) }} Метрів від вас
      </v-card-subtitle>
    </v-card>
  </template>
</template>

<script>
import { GoogleMap, Marker } from "vue3-google-map";
import citiesArray from "@/res/cities.json";
import { faMapPin, faStreetView } from "@fortawesome/free-solid-svg-icons";
import axios from "axios";

const CORS_ANYWHHERE_URL = import.meta.env.VITE_CORS_ANYWHERE_URL;
const GMAPS_URL =
  "https://maps.googleapis.com/maps/api/place/nearbysearch/json";
const GMAP_HOTEL_ICON = {
  path: faMapPin.icon[4],
  scale: 0.06,
  anchor: {
    x: faMapPin.icon[0] / 2,
    y: faMapPin.icon[1],
  },
  fillColor: "#FF5252",
  fillOpacity: 1,
  strokeWeight: 2,
  strokeColor: "#000000",
};

citiesArray.sort((a, b) => a.title.localeCompare(b.title));

function distanceCalculation(position1, position2) {
  const R = 6371e3;
  const f1 = (position1.lat * Math.PI) / 180;
  const f2 = (position2.lat * Math.PI) / 180;
  const df = ((position2.lat - position1.lat) * Math.PI) / 180;
  const dl = ((position2.lng - position1.lng) * Math.PI) / 180;

  const a =
    Math.sin(df / 2) * Math.sin(df / 2) +
    Math.cos(f1) * Math.cos(f2) * Math.sin(dl / 2) * Math.sin(dl / 2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  const d = R * c;
  return d;
}

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
          y: faStreetView.icon[1],
        },
        fillColor: "#512DA8",
        fillOpacity: 1,
        strokeWeight: 2,
        strokeColor: "#000000",
      },
    },
    hotels: [],
    closest: null,
  }),
  methods: {
    async defineLocation(event) {
      // Show Markers
      this.showMarkers = true;

      // Place User Marker
      const latLng = { lat: event.latLng.lat(), lng: event.latLng.lng() };
      let userMarkerCopy = { ...this.userMarker };
      userMarkerCopy.position = latLng;
      this.userMarker = userMarkerCopy;

      const config = {
        method: "get",
        url: `${CORS_ANYWHHERE_URL}${GMAPS_URL}?location=${latLng.lat}%2C${
          latLng.lng
        }&rankby=distance&type=lodging&key=${import.meta.env.VITE_GMAPS_KEY}`,
        headers: {},
      };
      let response = await axios(config);
      let hotels = response.data.results;
      hotels.filter((hotel) => {
        hotel.business_status === "OPERATIONAL";
      });
      hotels = [hotels[0], hotels[4], hotels[9], hotels[14], hotels[19]];
      this.hotels = hotels.map((gMapHotel, index) => {
        const imgUrl =
          "https://archive.org/download/no-photo-available/no-photo-available.png";
        try {
          imgUrl = `${CORS_ANYWHHERE_URL}https://maps.googleapis.com/maps/api/place/photo?maxwidth=400&photo_reference=${gMapHotel?.photos[0].photo_reference}&key=${this.key}`;
        } catch {
          console.log("error");
        }
        let hotel = {
          id: gMapHotel.place_id,
          icon: { ...GMAP_HOTEL_ICON },
          name: gMapHotel.name,
          position: gMapHotel.geometry.location,
          distance: distanceCalculation(latLng, gMapHotel.geometry.location),
          imgUrl: imgUrl,
        };
        console.log(index);
        if (index === 0) {
          hotel.icon.fillColor = "#388E3C";
        }
        return hotel;
      });
      this.closest = this.hotels[0];
    },
  },
  watch: {
    select(newSelect, oldSelect) {
      if (newSelect.value !== oldSelect.value) {
        this.showMarkers = false;
        this.closest = null;
      }
    },
  },
};
</script>
