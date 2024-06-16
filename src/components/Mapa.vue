<script setup>
import * as L from "leaflet";
import { LeafletLayer } from "deck.gl-leaflet";
import { MapView } from "@deck.gl/core";
import { GeoJsonLayer } from "@deck.gl/layers";
import { onMounted, ref, watch } from "vue";
import "leaflet/dist/leaflet.css";
import "leaflet.tilelayer.colorfilter/src/leaflet-tilelayer-colorfilter.js";

import hechos_transito_bicicletas from "../assets/datos/hechos_transito_bicicletas.json";
import infraestructura_vial_ciclista from "../assets/datos/infraestructura-vial-ciclista.json";
import { HeatmapLayer } from "@deck.gl/aggregation-layers";

const capa_visible = ref("calor");
const total_accidentes = ref(hechos_transito_bicicletas.features.length),
  accidentes_infraestructura = ref(0);
const distancia_infraestructura = ref(11),
  arrastre = ref(false);
const map = ref();
const datos_heatmap = ref(
  hechos_transito_bicicletas.features.map((d) => ({
    COORDINATES: [d.properties.coordenada_y, d.properties.coordenada_x],
    WEIGHT: 1,
  }))
);
function arrastrando() {
  arrastre.value = true;
}
function detenerArrastre() {
  arrastre.value = false;
}

const layers = ref([
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
]);
const deckLayer = ref();

const zoom_level = ref(10);
onMounted(() => {
  map.value = L.map(document.getElementById("map"), {
    center: [19.3, -99.1],
    zoom: zoom_level.value,
  });

  L.tileLayer
    .colorFilter("https://{s}.tile.openstreetmap.fr/hot/{z}/{x}/{y}.png", {
      filter: [
        "brightness:90%",
        "contrast:100%",
        "invert:100%",
        "grayscale:100%",
      ],
      attribution:
        '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Tiles style by <a href="https://www.hotosm.org/" target="_blank">Humanitarian OpenStreetMap Team</a> hosted by <a href="https://openstreetmap.fr/" target="_blank">OpenStreetMap France</a>',
      minZoom: 8,
    })

    .addTo(map.value);

  const view = new MapView({
    repeat: true,
  });

  deckLayer.value = new LeafletLayer({
    views: [view],
    layers: getLayers(),
  });
  map.value.addLayer(deckLayer.value);

  map.value.on("zoom", () => {
    zoom_level.value = map.value.getZoom();
    console.log(zoom_level.value - 7);
    deckLayer.value.setProps({ layers: [] });

    setTimeout(
      () =>
        deckLayer.value.setProps({
          layers: getLayers(),
        }),
      50
    );
  });
});
watch(capa_visible, (nv) => {
  deckLayer.value.setProps({ layers: [] });

  setTimeout(
    () =>
      deckLayer.value.setProps({
        layers: getLayers(),
      }),
    100
  );
});
watch(arrastre, (nv) => {
  if (!nv) {
    deckLayer.value.setProps({ layers: [] });
    setTimeout(
      () =>
        deckLayer.value.setProps({
          layers: getLayers(),
        }),
      100
    );
  }
});
function getLayers() {
  accidentes_infraestructura.value = hechos_transito_bicicletas.features.filter(
    (d) =>
      d.properties.distancia_minima_infraestructura <
      distancia_infraestructura.value
  ).length;
  const layers = [
    new GeoJsonLayer({
      id: "infraestructura",
      data: infraestructura_vial_ciclista,
      // Styles
      stroked: false,
      filled: true,
      extruded: true,
      lineWidthScale: 20,
      lineWidthMinPixels: 2,
      getLineColor: [211, 210, 255, 200],
      getPointRadius: 100,
      getLineWidth: 1,
      getElevation: 30,
    }),
    ...[
      capa_visible.value == "puntos"
        ? new GeoJsonLayer({
            id: "hechos",
            data: hechos_transito_bicicletas,
            // Styles
            filled: true,
            pointRadiusMinPixels: 2,
            pointRadiusScale: 2,
            getPointRadius: (f) => (0 < +f.properties.ciclista_occiso ? 44 : 8),
            getLineColor: (d) => [
              255,
              255,
              255,
              0 < +d.properties.ciclista_occiso ? 255 : 0,
            ],

            getFillColor: (d) =>
              d.properties.distancia_minima_infraestructura <
              distancia_infraestructura.value
                ? [
                    246,
                    71,
                    64,
                    0 < +d.properties.ciclista_occiso
                      ? 220
                      : 20 * (zoom_level.value - 7),
                  ]
                : [
                    255,
                    231,
                    0,
                    0 < +d.properties.ciclista_occiso
                      ? 220
                      : 20 * (zoom_level.value - 7),
                  ],
          })
        : new HeatmapLayer({
            id: "heatmapLayer",
            data: datos_heatmap.value,
            getPosition: (d) => d.COORDINATES,
            getWeight: (d) => d.WEIGHT,
            aggregation: "SUM",
            threshold: 0.05,
            opacity: 0.9,
            maxZoom: 10,
            visible: false
          }),
    ],
  ];
  return layers;
}
</script>

<template>
  <div class="contenedor-controles">
    <div class="contenedor-inputs-radios">
      
      <label for="puntos" class="contenedor-radios"
        >Mapa de puntos<input
          type="radio"
          id="puntos"
          value="puntos"
          v-model="capa_visible"
      /> <span class="checkmark"></span></label>

      <label for="calor" class="contenedor-radios"
        >Mapa de calor
        <input type="radio" id="calor" value="calor" v-model="capa_visible"
      /> <span class="checkmark"></span></label>
    </div>
    <div class="contenedor-input-rango">
      <div v-if="capa_visible == 'puntos'" class="control-distancia">
        <label for="rango"
          >Distancia: {{ distancia_infraestructura }}
          {{ distancia_infraestructura == 1 ? "metro" : "metros" }}
        </label>
        <input
          type="range"
          id="rango"
          v-model="distancia_infraestructura"
          step="1"
          min="1"
          max="100"
          @mousedown="arrastrando"
          @mouseup="detenerArrastre"
        />
        <p style="margin-top:12px">
          De {{ total_accidentes.toLocaleString("en-US") }} accidentes,
          {{ accidentes_infraestructura.toLocaleString("en-US") }} ({{
            Math.round((1000 * accidentes_infraestructura) / total_accidentes) /
            10
          }}%) se han registrado a menos de {{ distancia_infraestructura }}
          {{ distancia_infraestructura == 1 ? "metro" : "metros" }} de la
          infraestructura ciclista
        </p>
      </div>
    </div>
  </div>
  <div id="map"></div>
  <div  style="max-width: 500px">
    <p class="nomenclatura infra" v-if="capa_visible == 'puntos'">
      <span class="simbolo" style="background: rgb(211, 210, 255)"></span>
      Infraestructura ciclista
    </p>
    <p class="nomenclatura" v-if="capa_visible == 'puntos'">
      <span class="simbolo" style="background: rgb(246, 71, 64)"></span>
      Accidentes a menos de {{ distancia_infraestructura }} metros de la
      infraestructura ciclista
    </p>

    <p class="nomenclatura" v-if="capa_visible == 'puntos'">
      <span class="simbolo" style="background: rgb(255, 231, 0)"></span>
      Accidentes a más de {{ distancia_infraestructura }} metros de la
      infraestructura ciclista
    </p>
    <p v-if="capa_visible == 'puntos'">
      Los círculos más grandes representan hechos en los que hubo al menos
      alguna persona ciclista occisa*
    </p>
    
  </div>
</template>

<style scoped lang="scss">
#map {
  height: 80vh;
  width: 100%;
  background: rgb(29, 29, 29);
}
.contenedor-controles{
  display: flex;
  justify-content: space-around;
  .contenedor-inputs-radios{
    min-width: 150px;
  }
  .contenedor-input-rango{

  }
}
p.nomenclatura {
  span.simbolo {
    display: inline-block;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    transform: translateY(2px);
    margin-right: 3px;
  }
  &.infra{
    span.simbolo {
    border-radius:0;
  }
  }
}
</style>
