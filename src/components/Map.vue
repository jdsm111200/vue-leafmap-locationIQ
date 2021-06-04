<template>
  <div>
    <h1>Leaft Map</h1>

    <v-row>
      <v-col>
        <div>Pais: {{ pais }}</div>
        <div>Departamento: {{ departamento }}</div>
        <div>Municipio: {{ municipio }}</div>
        <div>Codigo Postal: {{ codigoPostal }}</div>
        <div>Calle: {{ calle }}</div>
        <div>Colonia: {{ colonia }}</div>
        <div>Latitud: {{ coordinates.lat }}</div>
        <div>Longitud: {{ coordinates.lng }}</div>
      </v-col>
      <v-col>
        <div>
          <v-text-field
            label="Direccion"
            v-model="addressFind"
            @keydown.enter="fowardGeocoding"
          ></v-text-field>
          {{ marker }}
          {{ addressFind }}
          <l-map
            :zoom="zoom"
            :center="center"
            @click="updateMarker"
            style="z-index: 0; height: 70vh; width: 70vh"
          >
            <l-tile-layer :url="url" :attribution="attribution" />
            <l-marker :lat-lng="marker" :icon="defaultMarkerIcon"></l-marker>
          </l-map>
        </div>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import { LMap, LTileLayer, LMarker, LPopup, LTooltip } from "vue2-leaflet";
import markerIconPng from "leaflet/dist/images/marker-icon.png";
import { latLng, Icon } from "leaflet";
import axios from "axios";
export default {
  components: { LMap, LTileLayer, LMarker, LPopup, LTooltip },
  data() {
    return {
      address: {},
      addressFind: "",
      calle: "",
      pais: "",
      departamento: "",
      municipio: "",
      codigoPostal: "",
      colonia: "",
      coordinates: { lat: "", lng: "" },
      zoom: 15,
      center: latLng(0, 0),
      url: "http://{s}.tile.osm.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      marker: latLng(0, 0),
      defaultMarkerIcon: new Icon({
        iconUrl: markerIconPng,
        iconSize: [25, 41],
        iconAnchor: [12, 41],
      }),
    };
  },
  async created() {
    var coord = await this.$getLocation({});
    this.marker = latLng(coord.lat, coord.lng);
    this.center = latLng(coord.lat, coord.lng);
    this.coordinates.lat = coord.lat;
    this.coordinates.lng = coord.lng;
  },
  methods: {
    async updateMarker(event) {
      this.coordinates.lat = event.latlng.lat;
      this.coordinates.lng = event.latlng.lng;
      this.marker = latLng(this.coordinates.lat, this.coordinates.lng);
      await this.reverseGeocoding();
    },
    async reverseGeocoding() {
      var res = await axios.get(
        "https://us1.locationiq.com/v1/reverse.php?key=pk.cb6782b8b417bd7495da5c64a0c8780f&lat=" +
          this.coordinates.lat +
          "&lon=" +
          this.coordinates.lng +
          "&format=json"
      );
      this.address = res.data.address;
      this.camposdeDireccion();
    },
    async fowardGeocoding() {
      var res = await axios.get(
        "https://us1.locationiq.com/v1/search.php?key=pk.cb6782b8b417bd7495da5c64a0c8780f&q=" +
          encodeURI(this.addressFind) +
          "&limit=1&format=json"
      );
      this.coordinates.lat = res.data[0].lat;
      this.coordinates.lng = res.data[0].lon;
      this.marker = latLng(this.coordinates.lat, this.coordinates.lng);
      this.center = latLng(this.coordinates.lat, this.coordinates.lng);
      await this.reverseGeocoding();
    },
    camposdeDireccion() {
      this.pais = this.address.country;
      this.departamento = this.address.state;
      this.municipio = this.address.county;
      this.calle = this.address.road;
      this.codigoPostal = this.address.postcode;
      this.colonia = "";
      delete this.address.country;
      delete this.address.state;
      delete this.address.county;
      delete this.address.road;
      delete this.address.country;
      delete this.address.postcode;
      delete this.address.country_code;
      for (const prop in this.address) {
        this.colonia = this.colonia + ", " + this.address[prop];
      }
      this.colonia = this.colonia.substring(2);
    },
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.currentCenter = center;
    },
    showLongText() {
      this.showParagraph = !this.showParagraph;
    },
    innerClick() {
      alert("Click!");
    },
  },
};
</script>

<style scoped>
.map {
  height: 95vh;
  z-index: 1;
}
</style>
