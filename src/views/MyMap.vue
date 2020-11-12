
<template>
  <div class="MyMap">
    <h1>Hello Map</h1>
    <div id="mapContainer" class="basemap"></div>
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
import getCoordinatesFromGpxFile from "@/modules/gpx-utilities.js";

export default {
  name: "MyMap",
  components: {
    Map,
  },
  data: function () {
    return {
      accessToken:
        "pk.eyJ1Ijoid2VpYmVscGF1bGEiLCJhIjoiY2toM2VqazdvMDVybjJ4bnl1NG82ZXZoMyJ9.Zg_IioTXydrTRsz8sYHnCA",
      commutes: [],
    };
  },

  //================================================
  // Loading Map
  //================================================
  created: async function () {
    console.log("created a module MyMap");
    let result = await contentfulClient.getEntries({ content_type: "person" });
    this.commutes = result.items;
  },
  mounted: function () {
    mapboxgl.accessToken = this.accessToken;

   var map = new mapboxgl.Map({
      container: "mapContainer",
      style: "mapbox://styles/weibelpaula/ckh8duvzo1apc19s7cjjmpcr0",
      center: [8.298254, 47.085445],
      zoom: 13,
      maxBounds: [
        [8.25739, 47.072158],
        [8.318511, 47.093011],
        /* [103.6, 1.1704753],
        [104.1, 1.4754753],*/
      ],
    });
  
    //===========================================
    //Loading GPX Coordinates
    //===========================================


    //===========================================
    //Creating GPX Trail
    //===========================================
     // Displaying a GPX track
    // Displaying a GPX track
    map.on("load", async function() {
      let result = await contentfulClient.getEntries({
        content_type: "gpx"
      });
      console.log(result.items);
      let coordinates = await getCoordinatesFromGpxFile(
        result.items[0].fields.gpxFile.fields.file.url // a link to you gpx file in Contentful
      );

      console.log(coordinates);
      
      map.addSource("route", {
        type: "geojson",
        data: {
          type: "Feature",
          geometry: {
            type: "LineString",
            coordinates: coordinates
          }
        }
      });
      map.addLayer({
        id: "route",
        type: "line",
        source: "route",
        layout: {
          "line-join": "round",
          "line-cap": "round"
        },
        paint: {
          "line-color": "#61ccc7",
          "line-width": 5
        }
      });
    });
  }
};





</script>


<style scoped>
@import url("https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css");
#mapContainer {
  position: fixed;
  top: 0px;
  left: 0px;
  z-index: 1;
  height: 100%;
  width: 100%;
}
#route {
  z-index: 10;
}
</style>