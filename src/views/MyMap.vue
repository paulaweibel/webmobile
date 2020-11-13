
<template>
  <div class="MyMap">
    <h1>Hello Map</h1>
    <div id="mapContainer" class="basemap"></div>
    <div id="spotlight" class="light"></div>
    <div id="story"></div>
    <li v-for="commute in commutes" :key="commute.fields">
      <Map
        :person="commute.fields.name"
        :img="commute.fields.picture.fields.file.url"
        :location="commute.fields.whereAmI.lat"
      />
    </li>
  </div>
</template>


<script defer>
let storypart = 0;

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
      zoom: 14,
      maxBounds: [
        [8.25739, 47.072158],
        [8.318511, 47.093011],
      ],
    });

    //===========================================
    //Creating GPX Trail
    //===========================================
    // Displaying a GPX track
    map.on("load", async function () {
      let result = await contentfulClient.getEntries({
        content_type: "gpx",
      });

      let coordinates = await getCoordinatesFromGpxFile(
        result.items[0].fields.gpxFile.fields.file.url // a link to you gpx file in Contentful
      );

      map.addSource("route", {
        type: "geojson",
        data: {
          type: "Feature",
          geometry: {
            type: "LineString",
            coordinates: coordinates,
          },
        },
      });
      map.addLayer({
        id: "route",
        type: "line",
        source: "route",
        layout: {
          "line-join": "round",
          "line-cap": "round",
        },
        paint: {
          "line-color": "#d3cdce",
          "line-width": 5,
        },
      });

      //===========================================
      //geojson markers from mapbox tutorial
      //===========================================
      //geojson from mapbox tutorial
      var geojson = {
        type: "FeatureCollection",
        features: [
          {
            type: "Feature",
            geometry: {
              type: "Point",
              coordinates: [8.298254, 47.085445],
            },
            properties: {
              title: "Mapbox",
              description: "Center Map",
            },
          },
          {
            type: "Feature",
            geometry: {
              type: "Point",
              coordinates: coordinates[100],
            },
            properties: {
              title: "Mapbox",
              description: "Another Point",
            },
          },
        ],
      };

      //===========================================
      //Creating Markers
      //===========================================
      let markers = [];
      let marker;
      for (let i = 0; i <= 3; i++) {
        markerNormal(i);
      }
      /*
      var marker = new mapboxgl.Marker({ color: "#ff0000" }).setLngLat(
        coordinates[0]
      );
      marker.getElement().setAttribute("id", "marker");
      marker.setPopup(new mapboxgl.Popup().setHTML("<h1>Hello World!</h1>"));
      marker.addTo(map);


      
      markers.forEach(element => element.getElement().addEventListener("mouseenter", markerHover(element)));
      */

      //===========================================
      //Marker Event Listeners and Functions
      //===========================================
      result = await contentfulClient.getEntries({
        content_type: "character",
      });

      //===========================================
      //Adding Geojson markers from tutorial
      //===========================================
      // add markers to map
      geojson.features.forEach(function (marker) {
        // create a HTML element for each feature
        var el = document.createElement("div");
        el.className = "marker";
        el.style.width = "50px";
        el.style.height = "50px";
        el.style.borderRadius = "50%";
        el.style.position = "absolute";
        el.style.zIndex = "4";

        // make a marker for each feature and add to the map
        let mark = new mapboxgl.Marker(el).setLngLat(
          marker.geometry.coordinates
        );

        let url =
          "url( http:" +
          result.items[0].fields.pictureofcharacter.fields.file.url +
          ")";
        console.log(url);
        mark.getElement().style.backgroundImage = url;
        mark.addTo(map);
        console.log(mark);
        console.log("should've loaded markers");
      });

      //===========================================
      //My marker methods
      //===========================================
      function markerHover(i) {
        console.log("hovering over marker " + i);
        markers[i].remove();
        marker = new mapboxgl.Marker({ color: "#ff9f51" }).setLngLat(
          coordinates[i * 15]
        );
        marker.getElement().setAttribute("id", "marker");
        markers[i] = marker;
        markers[i].addTo(map);
        let url =
          "url(" +
          result.items[0].fields.pictureofcharacter.fields.file.url +
          ")";

        markers[i].getElement().style.backgroundImage = url;
        markers[i].getElement().addEventListener("mouseleave", function () {
          markerNormal(i);
        });
        markers[i].getElement().addEventListener("click", function () {
          changeView(i);
        });
      }

      function markerNormal(i) {
        if (markers[i] != null) markers[i].remove();
        marker = new mapboxgl.Marker({ color: "#d10050" }).setLngLat(
          coordinates[i * 15]
        );
        marker.getElement().setAttribute("id", "marker");
        markers[i] = marker;
        markers[i].addTo(map);
        markers[i].getElement().addEventListener("mouseenter", function () {
          markerHover(i);
        });
      }

//Adding Elements to story div
      function changeView(i) {
        moveflag = false;
        let backgrounddiv = document.createElement("div");
        document.getElementById("story").appendChild(backgrounddiv);
        backgrounddiv.setAttribute("id", "character");
        console.log(backgrounddiv);
        if (i == storypart) {
          backgrounddiv.style.backgroundImage = `url( ${result.items[i].fields.pictureofcharacter.fields.file.url} )`;
          backgrounddiv.style.height = "100%";
          backgrounddiv.style.width = "100%";
          backgrounddiv.style.backgroundColor = "black";
          backgrounddiv.style.backgroundPosition = "bottom left";
          backgrounddiv.style.zIndex = "20";
          backgrounddiv.style.position = "absolute";
          backgrounddiv.style.backgroundRepeat = "no-repeat";
          backgrounddiv.style.top = "0px";
          backgrounddiv.style.left = "0px";
          
          console.log("story time");
        } else {
          console.log("wrong story part");
        }
        backgrounddiv.addEventListener("click", function () {
          console.log("can move again")
          let story = document.getElementById("story");
          story.removeChild(story.lastChild);
          storypart++;
          console.log(storypart);
          moveflag = true;
        });
      }
    });
  },
};
let moveflag = true;
//===========================================
//Spotlight
//===========================================

//Dont know why this spotlight method didn't work

window.addEventListener("mousemove", (e) => {
  if(moveflag == true){
  spotlightMove(e);
  }
});
function spotlightMove(e) {
  
    let string =
    "radial-gradient(circle at " +
    Math.round((e.pageX / window.innerWidth) * 100) +
    "% " +
    Math.round((e.pageY / window.innerHeight) * 100) +
    "%,transparent 160px,rgba(0, 0, 0, 0.89) 200px)";
  document.getElementById("spotlight").style.backgroundImage = string;

  
  
}
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

#spotlight {
  z-index: 5;
  position: absolute;
  background-image: radial-gradient(
    circle at 20% 20%,
    transparent 160px,
    rgba(0, 0, 0, 0.85) 200px
  );
  height: 100%;
  width: 100%;
  top: 0px;
  left: 0px;
  pointer-events: none;
}

#route {
  z-index: 10;
}

.marker {
  background-image: "..\assets\police-logo.png";
  background-size: cover;
  background-color: black;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  cursor: pointer;
  z-index: 4;
}

.mapboxgl-marker {
  width: 25px;
  height: 25px;
  border-radius: 50%;
  border: 1px solid gray !important;
  background-color: lightblue !important;
}

#character {
  background-color: black;
  height: 100%;
  width: 100%;
  background-size: 50%;
  bottom: 0px;
  left: 0px;
  position: absolute;
  z-index: 20;
}
</style>