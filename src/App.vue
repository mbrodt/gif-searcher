<template>
  <div id="app">
    <div class="container">

      <img src="./assets/giphy.gif">
      <h1>{{ msg }}</h1>
      <input type="text" class="search" v-model="searchInput" @keyup.enter="prepareUrl(false, searchInput)">
      <div class="btn-container">
        <button class="search-btn" @click="prepareUrl(false, searchInput)">Search!</button>
        <div>
          <label for="dropdown">Amount of gifs to display:</label>
          <select id="dropdown" v-model.number="amountToShow" @change="changeAmount()">
            <option value="3">3</option>
            <option value="9" selected>9</option>
            <option value="30">30</option>
          </select>
          <button @click="prepareUrl(true, '')">Random</button>
          <button @click="clear()">Clear</button>
        </div>
      </div>
      <p v-if="isLoading">Loading... </p>
      <div class="singlegif-mask" v-if="gifClicked" @click="closeGif()">
        <div class="singlegif-wrapper">
          <img class="singlegif-image" :src="gifClickedUrl" alt="">
        </div>
      </div>
      <div class="gif-container">
        <img class="random-img" v-if="isRandom" :src="randomGifUrl" @click="openGif(randomGifUrl)" alt="">
        <img class="searched-img" v-for="gif in gifsToShow" :src="gif" alt="" :key="gif" @click="openGif(gif)">
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
      searchUrl: "https://api.giphy.com/v1/gifs/search?",
      randomUrl: "https://api.giphy.com/v1/gifs/random?",
      searchInput: "",
      gifLinks: [],
      gifsToShow: [],
      randomGifUrl: "",
      gifClickedUrl: "",
      gifClicked: false,
      isRandom: false,
      isLoading: false,
      amountToShow: 9
    };
  },

  methods: {
    prepareUrl(random, searchTerm) {
      this.toggleLoading();
      let url = "";
      if (random) {
        this.isRandom = true;
        this.amountToShow = 1;
        url = `${this.randomUrl + this.apiKey}`;
      } else {
        this.amountToShow = 9;
        this.isRandom = false;
        let limit = 30;
        url = `${this.searchUrl + this.apiKey}&q=${searchTerm}&limit=${limit}`;
      }
      this.fetchGifs(url);
      this.searchInput = "";
    },
    fetchGifs(url) {
      fetch(url)
        .then(response => {
          return response.json();
        })
        .then(json => {
          this.buildGif(json);
          this.toggleLoading();
        })
        .catch(function(err) {
          console.log("Fetch Error :-S", err);
        });
    },

    buildGif(json) {
      this.clear();
      if (this.isRandom) {
        this.randomGifUrl = `https://media.giphy.com/media/${
          json.data.id
        }/giphy.gif`;
      } else {
        this.gifLinks = json.data
          .map(gif => gif.id)
          .map(gifId => `https://media.giphy.com/media/${gifId}/giphy.gif`);
        this.changeAmount();
      }
    },
    toggleLoading() {
      this.isLoading = !this.isLoading;
    },
    clear() {
      this.gifLinks = [];
      this.gifsToShow = [];
      this.randomGifUrl = "";
    },
    changeAmount() {
      this.gifsToShow = [];
      if (this.isRandom) {
        this.amountToShow = 1;
        return;
      }
      for (let i = 0; i < this.amountToShow; i++) {
        const element = this.gifLinks[i]; //could easily be randomized to show different orders
        this.gifsToShow.push(element);
      }
    },
    openGif(gif) {
      this.gifClickedUrl = gif;
      this.gifClicked = true;
      document.body.style.overflowY = "hidden";
    },
    closeGif() {
      this.gifClickedUrl = null;
      this.gifClicked = false;
      document.body.style.overflowY = "auto";
    }
  },
  mounted() {
    document.addEventListener("keydown", e => {
      if (this.gifClicked && e.keyCode == 27) {
        this.closeGif();
      }
    });
  }
};
</script>

<style lang="scss">

</style>


