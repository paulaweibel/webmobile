
<template>
  <div class="MyMap">
    <h1>Hello Map</h1>
    <div id="mapContainer" class="basemap"></div>
    <div id="spotlight" class="light"></div>
    <div id="story">
      <div id="character">
      </div>
      
        <p id="storytext"></p>
    </div>
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
let imgCounter = 0; //is used to switch between two images
let character = document.getElementById("character");
let storyEnd = true;
let storyCounter = 0;
let moveflag = true;

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

      //===========================================
      //Marker Event Listeners and Functions
      //===========================================
      result = await contentfulClient.getEntries({
        content_type: "storyline",
      });

      //===========================================
      //Adding Geojson markers from tutorial

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
          result.items[0].fields.img[0].fields.file.url +
          ")";
        console.log(url);
       // mark.getElement().style.backgroundImage = url;
        mark.getElement().style.zIndex = "15";
        mark.addTo(map);
        console.log(mark);
        console.log("should've loaded markers");
      });

      //===========================================
      //My marker methods
      //===========================================
      //changes marker when hovering over it
      function markerHover(i) {
        //console.log("hovering over marker " + i);
        markers[i].remove(); //delets old marker
        marker = new mapboxgl.Marker({ color: "#ff9f51" }).setLngLat(
          coordinates[i * 15]
        );
        marker.getElement().setAttribute("id", "marker");
        markers[i] = marker;
        markers[i].addTo(map); //adds new marker with the hover colour
        /*let url =
          "url(" +
          result.items[0].fields.img.fields.file.url +
          ")";

        markers[i].getElement().style.backgroundImage = url;
        */
        markers[i].getElement().addEventListener("mouseleave", function () {
          markerNormal(i);
        });
        markers[i].getElement().addEventListener("click", function () {
          changeView(i);
        });
      }
      //the marker when nothing is hovering over it
      function markerNormal(i) {
        if (markers[i] != null) markers[i].remove(); //only tries to delete when there is a marker to delete
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

      //===========================================
      //Story Methods
      //===========================================
      //Adding Elements to story div

    

      function changeView(i) {
        storyEnd = false;
        console.log(i, storypart);
        if (i === storypart) {
          character = document.getElementById("character");
          console.log("wtf, it gets stuck here and I don't know why")
          moveflag = false;
          character.style.zIndex = "25";
          console.log("url of background " + result.items[0].fields.background[0].fields.file.url);
          //console.log("trying to change zindex");
          //console.log(character.style.getPropertyValue("z-index"));
          //console.log(character);
          document.getElementById("story").style.zIndex = "20";
          storyTelling(i);

          character.addEventListener("click", function () {
            if (storyEnd == true) {
              console.log("can move again");
              console.log(storypart);
              storyCounter = 0;
            } else {
              storyTelling(i);
            }
          });
        } else {
          console.log("wrong story part");
        }
      }

      function storyTelling(i) {
       // console.log("trying to tell a story");
        //console.log(document.getElementById("story"))
        let textField = document.getElementById("storytext");
        //splitting text into seperate Elements
        let fullText = result.items[i].fields.text.content[0].content[0].value;
        let textElements;
        textElements = fullText.split("@");
        //console.log(textElements);
        //getting images from contentful
        let imgs = result.items[i].fields.img;
        //console.log(imgs)
        //console.log(storyCounter)

        character = document.getElementById("character");
        textField.innerHTML = textElements[storyCounter];
        textField.style.zIndex = "21";
        if (textElements.length === imgs.length) {
          let url = "url("+ imgs[storyCounter].fields.file.url +")";
          character.style.backgroundImage = url ;
        } else {
          console.log(imgCounter)
          if (imgCounter <= 1 || textElements.length - 1 == storyCounter) {
            //console.log("this should be happening");
            let url = "url( http:"+ imgs[0].fields.file.url +")";
            //console.log(url)
            character.style.backgroundImage = url;
            //console.log(character);
            //console.log("after assigning");
            imgCounter++;
          } else if(imgCounter == 2 ){
            let url = "url( 'http:"+ imgs[1].fields.file.url +"')";
            character.style.backgroundImage = url;
            imgCounter--;
          }
        }
        if (textElements[storyCounter] === textElements[textElements.length - 1]) {
          console.log("end of story part")
          storyEnd = true;
          storyCounter = 0;
          imgCounter = 0;
          character.style.zIndex = "0";
          document.getElementById("story").style.zIndex = "0";
          textField.style.zIndex = "0";
          storypart++;
          moveflag = true;
          
        } else {
          storyCounter++;
        }
      }
    });
  },
};
//===========================================
//Spotlight
//===========================================

//Dont know why this spotlight method didn't work

window.addEventListener("mousemove", (e) => {
  if (moveflag == true) {
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
  background-color: transparent;
  background-repeat: no-repeat;
  background-position: bottom left;
  height: 100%;
  width: 100%;
  background-size: 50%;
  bottom: 0px;
  left: 0px;
  position: absolute;
  z-index: 0;
  background-image: unset;
}

#story{
  background-color: black;
  z-index: 0;
  height: 100%;
  width:100%;
  position:absolute;
  top:0px;
  left:0px;
}

#storytext {
  z-index: 0;
  display: block;
  position: absolute;
  width: 20%;
  top: 50%;
  right: 5%;
  text-align: center;
  color: azure;
}
</style>



<!--
console.log(backgrounddiv);
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
-->