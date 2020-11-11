

<template>
  <div class="commute">
    <h2>
      {{person}}'s commute
    </h2>
    <img :src="img" alt="Picture here" width="600px">
    <p>
      This is <span class="person">{{person}}</span>
    </p>
    <p>
      {{person}} commutes from {{whereAmI}}
    </p>
  </div>
</template>

<script>
export default {
  name: "Commute",
  props: {
    person: String,
    img: String,
    location: Number,
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">

</style>



<!--
<template>
  <div class="Commute">
      <h1>
          {{person}}'s Commute'
      </h1>
      <p>
          {{person}} commutes from Bar
      </p>
  </div>
</template>

<script>
export default {
  name: 'Commute',
  props: {
    person: String
  }
}
</script>

<style scoped>

</style>

-->