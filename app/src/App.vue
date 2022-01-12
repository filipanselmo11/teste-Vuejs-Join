<template>
  <v-app>
    <v-row>
      <v-col>
        <Header />
        <v-main>
          <v-col cols="12">
            <Filtro @stations="getStations" @types="getTypes" />
          </v-col>
          <v-card>
            <div id="map"></div>
          </v-card>
        </v-main>
        <v-card
          id="tooltipMap"
          width="420"
          height="415"
          class="text-center popup-container"
          outlined
        >
          <v-card-title class="headline cyan lighten-1">
            <v-row>
              <span style="color: white">Dados da Estação</span>
              <v-spacer></v-spacer>
              <v-btn @click="close" small icon color="white"
                ><v-icon>clear</v-icon></v-btn
              >
            </v-row>
          </v-card-title>
          <v-card-text class="popup-container">
            <v-container fluid class="popup-container">
              <v-row align="center" class="popup-container mb-n7">
                <v-col class="d-flex" cols="12">
                  <label style="font-size: large; font-weight: bold">ID: </label
                  ><strong
                    style="font-size: large; font-weight: bold"
                    class="overlay-text"
                    id="feature-id"
                  ></strong>
                </v-col>
                <v-col class="d-flex" cols="12">
                  <label style="font-size: large; font-weight: bold"
                    >Estação: </label
                  ><strong
                    style="font-size: large; font-weight: bold"
                    class="overlay-text"
                    id="feature-name"
                  ></strong>
                </v-col>
                <v-col class="d-flex" cols="12">
                  <v-icon>place</v-icon>
                  <label style="font-size: large; font-weight: bold"
                    >Latitude:</label
                  >
                  <strong
                    style="font-size: large; font-weight: bold"
                    class="overlay-text"
                    id="feature-latitude"
                  ></strong>
                </v-col>
                <v-col class="d-flex" cols="12">
                  <v-icon>place</v-icon>
                  <label style="font-size: large; font-weight: bold"
                    >Longitude:</label
                  ><strong
                    style="font-size: large; font-weight: bold"
                    class="overlay-text"
                    id="feature-longitude"
                  ></strong>
                </v-col>
                <v-col class="d-flex" cols="12">
                  <label style="font-size: large; font-weight: bold"
                    >Elevação:</label
                  ><strong
                    style="font-size: large; font-weight: bold"
                    class="overlay-text"
                    id="feature-elevation"
                  ></strong>
                </v-col>
                <v-col class="d-flex" cols="12">
                  <label style="font-size: large; font-weight: bold"
                    >Tipo de estação:</label
                  ><strong
                    style="font-size: large; font-weight: bold"
                    class="overlay-text"
                    id="feature-type"
                  ></strong>
                </v-col>
                <v-col class="d-flex" cols="12">
                  <v-icon>event</v-icon>
                  <label style="font-size: large; font-weight: bold"
                    >Inicio de operação:</label
                  ><strong
                    style="font-size: large; font-weight: bold"
                    class="overlay-text"
                    id="feature-start"
                  ></strong>
                </v-col>
                <v-col class="d-flex" cols="12">
                  <v-icon>event</v-icon>
                  <label style="font-size: large; font-weight: bold"
                    >Fim de operação:</label
                  ><strong
                    style="font-size: large; font-weight: bold"
                    class="overlay-text"
                    id="feature-end"
                  ></strong>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-app>
</template>

<script>
/* eslint-disable */
import "ol/ol.css";
import GeoJSON from "ol/format/GeoJSON";
import Map from "ol/Map";
import View from "ol/View";
import { defaults as defaultControls, ScaleLine } from "ol/control";
import { Tile as TileLayer, Vector as VectorLayer } from "ol/layer";
import { OSM, Vector as VectorSource } from "ol/source";
import { Style, Icon } from "ol/style";
import Overlay from "ol/Overlay";
import Filtro from "./components/Filtro.vue";
import Header from "./components/Header.vue";

export default {
  name: "App",
  components: {
    Filtro,
    Header,
  },
  data: () => ({
    station_type_id: undefined,
    types: undefined,
    stations: undefined,
    vectorGeoJSON: new VectorLayer(),
    source: new VectorSource({
      format: new GeoJSON(),
      url: "https://raw.githubusercontent.com/filipanselmo11/DadosJSON/master/data/station_list.geojson",
    }),
  }),
  async mounted() {
    await this.init();
  },
  methods: {
    close() {
      let tooltip = document.getElementById("tooltipMap");
      tooltip.style.visibility = "hidden";
      tooltip.style.display = "none";
    },
    async getTypes(types) {
      this.types = types;
    },

    async getStations(stations) {
      this.stations = stations;
      if (stations.features.length > 0 && stations.features.length < 125) {
        this.source.clear();
        this.source.addFeatures(new GeoJSON().readFeatures(stations));
      } else if (stations.features.length === 125) {
        this.source.refresh();
      } else if (stations.features.length === 0) {
        this.source.clear();
      }
    },

    init() {
      let self = this;
      var imageStyle = new Icon({
        scale: 1.0,
        src: require("./assets/location_on-white-48dp.svg"),
      });

      var source = new VectorSource();
      var vector = new VectorLayer({
        source: source,
      });

      var raster = new TileLayer({
        source: new OSM(),
      });

      this.vectorGeoJSON = new VectorLayer({
        source: this.source,
        style: function (feature, resolution) {
          return [
            new Style({
              image: new Icon({
                scale: 0.9,
                color:
                  feature.get("station_type_id") == "1"
                    ? "#7cb5ec"
                    : feature.get("station_type_id") == "2"
                    ? "#434348"
                    : feature.get("station_type_id") == "3"
                    ? "#90ed7d"
                    : feature.get("station_type_id") == "4"
                    ? "#f7a35c"
                    : "#8085e9",
                src: require("./assets/location_on-white-48dp.svg"),
              }),
            }),
          ];
        },
      });

      var map = new Map({
        controls: defaultControls().extend([
          new ScaleLine({
            units: "degrees",
          }),
        ]),
        target: "map",
        layers: [raster, vector, this.vectorGeoJSON],
        view: new View({
          projection: "EPSG:4326",
          center: [-51.815011395380765, -24.650150016322684],
          zoom: 7,
        }),
      });

      var popup = document.querySelector(".popup-container");
      var overlayLayer = new Overlay({
        element: popup,
      });
      map.addOverlay(overlayLayer);
      let station_type = [
        {
          id: "1",
          created_at: "2020-11-30 10:43:46.687141-03",
          update_at: "2020-11-30 10:43:46.687162-03",
          name: "AGROMETEOROLOGICAL",
          color: "#7cb5ec",
        },
        {
          id: "2",
          created_at: "2020-11-30 10:43:46.688442-03",
          update_at: "2020-11-30 10:43:46.688459-03",
          name: "CLIMATOLÓGICO",
          color: "#434348",
        },
        {
          id: "3",
          created_at: "2020-11-30 10:43:46.68895-03",
          update_at: "2020-11-30 10:43:46.688964-03",
          name: "HIDROCLIMATOLÓGICO",
          color: "#90ed7d",
        },
        {
          id: "4",
          created_at: "2020-11-30 10:43:46.689495-03",
          update_at: "2020-11-30 10:43:46.689513-03",
          name: "HIDROMÉTRICO",
          color: "#f7a35c",
        },
        {
          id: "5",
          created_at: "2020-11-30 10:43:46.690113-03",
          update_at: "2020-11-30 10:43:46.690134-03",
          name: "PLUVIOMETRIC",
          color: "#8085e9",
        },
      ];

      var overlayFeatureId = document.getElementById("feature-id");
      var overlayFeatureName = document.getElementById("feature-name");
      var overlayFeatureLatitude = document.getElementById("feature-latitude");
      var overlayFeatureLongitude =
        document.getElementById("feature-longitude");
      var overlayFeatureElevation =
        document.getElementById("feature-elevation");
      var overlayFeatureType = document.getElementById("feature-type");
      var overlayFeatureStart = document.getElementById("feature-start");
      var overlayFeatureEnd = document.getElementById("feature-end");

      map.on("click", function (e) {
        overlayLayer.setPosition(undefined);
        map.forEachFeatureAtPixel(e.pixel, function (feature, layer) {
          overlayLayer.setPosition(e.coordinate);
          overlayFeatureId.innerHTML = feature.get("id");
          overlayFeatureName.innerHTML = feature.get("name");
          overlayFeatureLatitude.innerHTML = feature.get("latitude");
          overlayFeatureLongitude.innerHTML = feature.get("longitude");
          overlayFeatureElevation.innerHTML = feature.get("elevation_meters");
          overlayFeatureStart.innerHTML = feature.get("operation_start_date");
          overlayFeatureEnd.innerHTML = feature.get("operation_end_date");
          for (let item in station_type) {
            if (feature.get("station_type_id") == station_type[item].id) {
              overlayFeatureType.innerHTML = station_type[item].name;
            }
          }
        });
        let tooltip = document.getElementById("tooltipMap");
        tooltip.style.visibility = "visible";
        tooltip.style.display = "block";
      });
    },
  },
};
</script>

<style>
#map {
  position: absolute;
  padding: 0;
  z-index: 0;
  margin-top: 0px;
  height: 1000px;
  width: 100%;
}
</style>