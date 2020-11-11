
<template>
  <div class="home">
    <li v-for="commute in commutes" :key="commute.fields">
      <Commute :person="commute.fields.name" :img="commute.fields.picture.fields.file.url" :location="commute.fields.whereAmI.lat" />
    </li>
  </div>
</template>

<script>
// @ is an alias to /src
import Commute from "@/components/Commute.vue";
import contentfulClient from "@/modules/contentful.js"

export default {
  name: "Home",
  components: {
    Commute
  },
  data: function() {
    return {
      commutes: []
    };
  },
  created: async function() {
    console.log("created a module Home");
    let result = await contentfulClient.getEntries(
      {content_type : "person"}
    );
    this.commutes = result.items;
    console.log(this.commutes);
  }
};
</script>




<!--
<template>
  <div class="home">
    <ul>
      <li v-for="commute in commutes" :key="commute.fields.title">
         <Commute :person="commute.fields.person.name" />
      </li>
    </ul>
  </div>
</template>

<script>
// @ is an alias to /src
import Commute from '@/components/Commute.vue'
import contentfulClient from "@/modules/contentful.js"
export default {
  name: 'Home',
  components: {
    Commute,
  },
  data: function(){
    return{
      commutes: ["Marc", "Lena", "Maria"]
    };
  },
  created: async function() {
    console.log("created a module Home");
    let result = await contentfulClient.getEntries(
      {content_type : "commute"}
    );
    this.commutes = result.items;
    console.log(this.commutes);
  }
};
</script>



-->





<!--
muster lösung


<template>
  <div class="home">
    <li v-for="commute in commutes" :key="commute.person">
      <Commute :person="commute.person" :start="commute.start" :destination="commute.destination" />
    </li>
  </div>
</template>

<script>
// @ is an alias to /src
import Commute from "@/components/Commute.vue";
export default {
  name: "Home",
  components: {
    Commute
  },
  data: function() {
    return {
      commutes: [
        {
          person: "Maria",
          start: "Baar",
          destination: "Emmenbrücke"
        },
        {
          person: "Hanna",
          start: "Zürich",
          destination: "Emmenbrücke"
        }
      ]
    };
  }
};
</script>

-->