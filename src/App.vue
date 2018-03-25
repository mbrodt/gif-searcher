<template>
  <div id="app">
    <div class="container">

    <img src="./assets/logo.png">
    <h1>{{ msg }}</h1>
    <input type="text" class="search" v-model="searchInput" @keyup.enter="fetchGif(searchInput)">
    <div >
      <button class="search-btn" @click="fetchGif(searchInput)">Search!</button>
      <button @click="gifLinks=[]">Clear</button>
      </div>
      <div class="gif-container">
        <img v-for="gif in gifLinks" :src="gif" alt="" :key="gif">
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      msg: "Super Awesome Gif Searcher",
      apiKey: "&api_key=lhFQfxgJLjCyauVDsuPp9tExkQoSN6gf",
      apiUrl: "https://api.giphy.com/v1/gifs/search?",
      searchInput: "",
      gifLinks: []
    };
  },

  methods: {
    fetchGif: function(searchTerm) {
      this.gifLinks = [];
      let limit = 9;
      const url = `${this.apiUrl + this.apiKey}&q=${searchTerm}&limit=${limit}`;
      fetch(url)
        .then(response => {
          return response.json();
        })
        .then(json => {
          this.gifLinks = json.data
            .map(gif => gif.id)
            .map(gifId => `https://media.giphy.com/media/${gifId}/giphy.gif`);
        })
        .catch(function(err) {
          console.log("Fetch Error :-S", err);
        });
    }
  }
};
</script>

<style lang="scss">
html {
  box-sizing: border-box;
}
*,
*:before,
*:after {
  box-sizing: inherit;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.gif-container {
  display: grid;
  grid-gap: 15px;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  img {
    height: 300px;
    width: 100%;
    object-fit: cover;
  }
}

h1,
h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

.search {
  padding-left: 1rem;
  font-size: 2rem;
  height: 3rem;
  width: 25rem;
}

.search-btn {
  width: 200px;
  height: 50px;
  margin-top: 2rem;
  margin-bottom: 2rem;
}
</style>
