<template class="main">
  <v-toolbar color="primary">
    <v-toolbar-title> Курсова робота</v-toolbar-title>
  </v-toolbar>
  
  <v-container fluid>
    <v-row align="center">
        <v-col cols="4">
            <v-subheader>
                Поточне місто
            </v-subheader>
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
  >
    <Marker :options="{ position: center }" />
  </GoogleMap>
</template>

<script>
import { GoogleMap, Marker } from "vue3-google-map";
import citiesArray from '@/res/cities.json';

citiesArray.sort((a, b) => a.title.localeCompare(b.title))

export default {
  components: { GoogleMap, Marker },
  data: () => ({
    key: process.env.GMAPS_KEY,
    center: { lat: 40.689247, lng: -74.044502 },
    select: { title: 'Київ', value: "KV", center: {lat: 50.448287, lng: 30.522876}, zoom: 11},
    cities: citiesArray,
  }),
};
</script>
