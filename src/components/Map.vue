
<template>
  <div class="map">
    
    <h2>
      {{person}}'s commute
    </h2>
   <!-- <img :src="img" alt="Picture here" width="600px"> -->
    <p>
      This is {{person}}
    </p>
  </div>
</template>

<script>
export default {
  name: "Map",
  props: {
    person: String,
    img: String,
  }
};


</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">

</style>


