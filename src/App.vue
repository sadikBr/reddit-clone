<template>
  <div id="app">
    <div class="nav">
      <h1 class="logo">Reddit</h1>
      <div class="links">
        <router-link to="/">Home</router-link>
        <router-link to="/image">About</router-link>
      </div>
    </div>
    <router-view />
  </div>
</template>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  width: 100vw;
  height: 100vh;
  color: #2c3e50;
  overflow-x: hidden;
}

#app {
  width: 100%;
  height: 100%;
  width: 80%;
  margin: auto;
}

.nav {
  display: flex;
  padding: 20px 0px;
  justify-content: space-between;
  align-items: center;
}

.logo {
  color: #42b983;
  flex: 5;
}

.links {
  display: flex;
  justify-content: space-between;
  flex: 1;

  a {
    font-weight: bold;
    text-transform: uppercase;
    text-decoration: none;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
