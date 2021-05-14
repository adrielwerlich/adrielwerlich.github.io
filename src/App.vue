<template>
  <v-app>
    <v-app-bar app
      style="justify-content: center; text-transform: uppercase;"
    > App de uso da api dos pokemons </v-app-bar>

    <v-main>
      <v-container>
        <h1>Pokemon List</h1>
        <hr>
        <router-view/>
      </v-container>
    </v-main>
  </v-app>
</template>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
