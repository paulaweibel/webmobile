
<template>
  <div class="MyMap">
    <li v-for="commute in commutes" :key="commute.fields">
      <Map :person="commute.fields.name" :img="commute.fields.picture.fields.file.url" :location="commute.fields.whereAmI.lat" />
    </li>
  </div>
</template>

<script>
// @ is an alias to /src
import Map from "@/components/Map.vue";
import contentfulClient from "@/modules/contentful.js"

export default {
  name: "MyMap",
  components: {
    Map
  },
  data: function() {
    return {
      commutes: []
    };
  },
  created: async function() {
    console.log("created a module MyMap");
    let result = await contentfulClient.getEntries(
      {content_type : "person"}
    );
    this.commutes = result.items;
    console.log(this.commutes);
  }
};
</script>

