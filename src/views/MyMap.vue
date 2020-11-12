
<template>
  <div class="MyMap">
    <h1>Hello Map</h1>
    <div id="mapContainer" class="basemap" ></div>
    <li v-for="commute in commutes" :key="commute.fields">
      <Map
        :person="commute.fields.name"
        :img="commute.fields.picture.fields.file.url"
        :location="commute.fields.whereAmI.lat"
      />
    </li>
  </div>
</template>


<script>
// @ is an alias to /src
import Map from "@/components/Map.vue";
import contentfulClient from "@/modules/contentful.js";
import mapboxgl from "mapbox-gl";



export default {
  name: "MyMap",
  components: {
    Map,
  },
  data: function () {
    return {
      accessToken: 'pk.eyJ1Ijoid2VpYmVscGF1bGEiLCJhIjoiY2toM2VqazdvMDVybjJ4bnl1NG82ZXZoMyJ9.Zg_IioTXydrTRsz8sYHnCA',
      commutes: [],
    };
  },
  created: async function () {
    console.log("created a module MyMap");
    let result = await contentfulClient.getEntries({ content_type: "person" });
    this.commutes = result.items;
    console.log(this.commutes);
  },
  mounted: function() {
    mapboxgl.accessToken = this.accessToken;

    new mapboxgl.Map({
      container: "mapContainer",
      style: "mapbox://styles/weibelpaula/ckh8duvzo1apc19s7cjjmpcr0",
      center: [8.298254, 47.085445],
      zoom: 13,
      maxBounds: [
        [8.257390, 47.072158],
        [8.318511, 47.093011],
       /* [103.6, 1.1704753],
        [104.1, 1.4754753],*/
      ],
    });
  },
};
</script>


<style scoped>
  @import url("https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css");
  #mapContainer{
    position: fixed;
    top: 0px;
    left: 0px;
    z-index: 1;
    height: 100%;
    width: 100%;
  }
</style>