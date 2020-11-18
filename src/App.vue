<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link> |
      <router-link to="/mymap">Map</router-link>
    </div>
    <router-view/>
  </div>
</template>

<style>
:root {
    --background: rgb(4, 31, 37);
    --content-background: rgba(21, 21, 21, 0.6);
    --header: rgba(130, 155, 252, 0.185);
    --footer: rgb(20, 38, 54);

    --title: rgb(121, 252, 210);
    --text: rgb(184, 218, 207);
    --something: rgb(47, 115, 66);

}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color:var(--text);
}

#nav {
  padding: 30px;
}

#nav a {
  position: relative;
  z-index: 10;
  font-weight: bold;
  color: var(--text);
}

#nav a.router-link-exact-active {
  color: #42b983;
}

.body{
  background-color: black;
}
</style>
