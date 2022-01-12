<template>
  <v-container fluid>
    <v-row class="text-center">
      <v-col cols="12" sm="12" md="12">
        <v-container fluid>
          <v-row>
            <v-card min-width="1683" max-width="1900">
              <v-card-title class="headline cyan lighten-1">
                <span style="color: white">Filtros</span>
              </v-card-title>
              <v-card-text>
                <br />
                <v-select
                  v-model="stationsTypesSelected"
                  :items="stationTypeList"
                  return-object
                  item-text="name"
                  label="Tipos"
                  multiple
                  outlined
                  chips
                  clearable
                  item-color="cyan lighten-1"
                >
                  <template v-slot:prepend-item>
                    <v-list-item ripple @click="change">
                      <v-list-item-action>
                        <v-icon color="cyan lighten-1" dark>
                          {{ typesIcon }}
                        </v-icon>
                      </v-list-item-action>
                      <v-list-item-content>
                        <v-list-item-title>
                          Selecionar Todos
                        </v-list-item-title>
                      </v-list-item-content>
                    </v-list-item>
                  </template>

                  <template v-slot:selection="data">
                    <v-chip
                    color="cyan lighten-1"
                    dark
                      :key="JSON.stringify(data.item)"
                      v-bind="data.attrs"
                      :input-value="data.selected"
                      :disabled="data.disabled"
                      @click:close="data.parent.selectItem(data.item)"
                    >
                      <strong>{{ data.item.name }}</strong>
                    </v-chip>
                  </template>
                </v-select>

                <v-select
                  v-model="stations_selected"
                  :items="stations"
                  return-object
                  item-key="id"
                  item-text="name"
                  label="Estações"
                  multiple
                  outlined
                  chips
                  clearable
                  item-color="cyan lighten-1"
                  :disabled="!stations_types_selected.length > 0"
                >
                  <template v-slot:no-data>
                    <v-list-item>
                      <strong style="color: red"
                        >Sem tipo de estações selecionado</strong
                      >
                    </v-list-item>
                  </template>

                  <template v-slot:prepend-item>
                    <v-list-item ripple @click="toggle">
                      <v-list-item-action>
                        <v-icon color="cyan lighten-1" dark>
                          {{ stationsIcon }}
                        </v-icon>
                      </v-list-item-action>
                      <v-list-item-content>
                        <v-list-item-title>
                          Selecionar Todos
                        </v-list-item-title>
                      </v-list-item-content>
                    </v-list-item>
                    <v-divider />
                  </template>

                  <template v-slot:selection="data">
                    <v-chip
                    color="cyan lighten-1"
                    dark
                      :key="JSON.stringify(data.item)"
                      v-bind="data.attrs"
                      :input-value="data.selected"
                      :disabled="data.disabled"
                      @click:close="data.parent.selectItem(data.item)"
                    >
                      <strong>{{ data.item.name }}</strong>
                    </v-chip>
                  </template>
                </v-select>
                <span
                  v-show="stationsSelected.length === 0"
                  style="color: red"
                  >Selecione pelo menos 1 tipo de estação e pelo menos 1 estação</span
                >
              </v-card-text>
              <v-divider></v-divider>
              <v-card-actions>
                <v-spacer />
                <v-btn :disabled="stationsSelected.length === 0" dark color="cyan lighten-1" @click="searching">Pesquisar</v-btn>
              </v-card-actions>
            </v-card>
          </v-row>
        </v-container>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
/* eslint-disable */
export default {
  data: () => ({
    stationList: [],
    stationTypeList: [],
    stationsTypesSelected: [],
    stationsSelected: [],
    geojson: undefined,
  }),

  created() {
    this.trackStation();
    this.trackStationType();
    this.trackFeatures();
  },

  computed: {
    stations() {
      let ax = [];
      if (this.stationsTypesSelected.length !== 0) {
        for (let station in this.stationsTypesSelected) {
          let fil = this.stationList.filter((a) => {
            return (
              a.stationTypeId === this.stationsTypesSelected[station].id
            );
          });
          for (let item in fil) {
            ax.push(fil[item]);
          }
        }
        return ax;
      } else {
        return [];
      }
    },

    allTypes() {
      return (
        this.stationsTypesSelected.length === this.stationTypeList.length
      );
    },

    determinedTypes() {
      return (
        this.stationsTypesSelected.length > 0 &&
        this.stationsTypesSelected.length < this.stationTypeList.length
      );
    },

    cleanTypes() {
      return this.stationsTypesSelected.length === 0;
    },

    allStations() {
      return this.stationsSelected.length === this.stations.length;
    },

    determinedStations() {
      return (
        this.stationsSelected.length > 0 &&
        this.stationsSelected.length < this.stations.length
      );
    },

    cleanStations() {
      return this.stationsSelected.length === 0;
    },

    stationsIcon() {
      if (this.allStations) {
        return "disabled_by_default";
      }

      if (this.determinedStations) {
        return "indeterminate_check_box";
      }

      if (this.cleanStations) {
        return "check_box_outline_blank";
      }
    },

    typesIcon() {
      if (this.allTypes) {
        return "disabled_by_default";
      }

      if (this.determinedStations) {
        return "indeterminate_check_box";
      }

      if (this.cleanTypes) {
        return "check_box_outline_blank";
      }
    },
  },

  methods: {
    change(){
      this.$nextTick(() => {
        if (this.allTypes) {
          this.stationsTypesSelected = []
        }
        else if(this.determinedTypes){
          this.stationsTypesSelected = []
        } else if (this.cleanTypes) {
          this.stationsTypesSelected = this.stationTypeList.slice()
        }
      })
    },
    toggle() {
      this.$nextTick(() => {
        if (this.allStations) {
          this.stationsSelected = [];
        } else if (this.determinedStations) {
          this.stationsSelected = [];
        } else if (this.cleanStations) {
          this.stationsSelected = this.stations.slice();
        }
      });
    },

    trackFeatures() {
      fetch(
        "https://raw.githubusercontent.com/filipanselmo11/DadosJSON/master/data/station_list.geojson"
      )
        .then((response) => response.json())
        .then((response) => {
          this.geojson = response;
        })
        .catch((e) => {
          console.log("Error: ", e);
        });
    },

    trackStation() {
      fetch(
        "https://raw.githubusercontent.com/filipanselmo11/DadosJSON/master/data/station.json"
      )
        .then((response) => response.json())
        .then((response) => {
          this.stationList = response.station;
        })
        .catch((e) => {
          console.log("Error: ", e);
        });
    },

    trackStationType() {
      fetch(
        "https://raw.githubusercontent.com/filipanselmo11/DadosJSON/master/data/station_type.json"
      )
        .then((response) => response.json())
        .then((response) => {
          this.stationTypeList = response.station_Type;
        })
        .catch((e) => {
          console.log("Error: ", e);
        });
    },

    searching() {
      let features = [];
      for (let item in this.geojson.features) {
        for (let station in this.stationsSelected) {
          if (
            this.geojson.features[item].properties.id ===
            this.stationsSelected[station].id
          ) {
            features.push(this.geojson.features[item]);
          }
        }
      }
      this.geojson.features = features;
      this.$emit("stations", this.geojson);
      this.$emit("types", this.stationsTypesSelected);
      this.trackFeatures();
    },
  },
};
</script>

<style>
</style>