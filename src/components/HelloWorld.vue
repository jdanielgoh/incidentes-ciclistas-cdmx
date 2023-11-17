<script setup>
import * as L from "leaflet";
import { LeafletLayer } from "deck.gl-leaflet";
import { MapView } from "@deck.gl/core";
import { GeoJsonLayer } from "@deck.gl/layers";
import { onMounted } from "vue";
import "leaflet/dist/leaflet.css";
import "leaflet.tilelayer.colorfilter/src/leaflet-tilelayer-colorfilter.js";

import hechos_transito_bicicletas from "../assets/datos/hechos_transito_bicicletas.json";
import infraestructura_vial_ciclista from "../assets/datos/infraestructura-vial-ciclista.json";
import { HeatmapLayer } from "@deck.gl/aggregation-layers";

onMounted(() => {
  // source: Natural Earth http://www.naturalearthdata.com/ via geojson.xyz

  const map = L.map(document.getElementById("map"), {
    center: [19.37, -99],
    zoom: 10,
  });
  L.tileLayer
    .colorFilter("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      filter: ["saturation:1%", "invert:100%"],

      attribution:
        '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
    })
    .addTo(map);

    const datos_heatmap= hechos_transito_bicicletas.features.map(d=>({"COORDINATES":[d.properties.coordenada_y, d.properties.coordenada_x], WEIGHT:1}))
  const deckLayer = new LeafletLayer({
    views: [
      new MapView({
        repeat: true,
      }),
    ],
    layers: [
      new GeoJsonLayer({
        id: "infraestructura",
        data: infraestructura_vial_ciclista,
        // Styles
        stroked: false,
        filled: true,
        extruded: true,
        pointType: "circle",
        lineWidthScale: 20,
        lineWidthMinPixels: 2,
        getLineColor: [160, 10, 180, 200],
        getPointRadius: 100,
        getLineWidth: 1,
        getElevation: 30,
      }),
      new GeoJsonLayer({
        id: "hechos",
        data: hechos_transito_bicicletas,
        // Styles
        filled: true,
        pointRadiusMinPixels: 2,
        pointRadiusScale: 20,
        getPointRadius: (f) => 5 - f.properties.scalerank,
        getFillColor: [250, 250, 80, 160],
      }),
      new HeatmapLayer({
        id: "heatmapLayer",
        data: datos_heatmap,
        getPosition: (d) => d.COORDINATES,
        getWeight: (d) => d.WEIGHT,
        aggregation: "SUM",
        threshold: 0.05,
        opacity: 0.4,

      }),
    ],
  });
  map.addLayer(deckLayer);


});
</script>

<template>
  <div id="map"></div>
</template>

<style scoped>
#map {
  height: 100vh;
  width: 100%;
  background: rgb(29, 29, 29);
}
</style>
