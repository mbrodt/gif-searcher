<template>
  <div id="app" class="font-sans text-center antialiased text-blue-darkest mt-8">
    <div class="container">

      <img src="./assets/giphy.gif">
      <h1 class="font-semibold my-6">{{ msg }}</h1>
      <input type="text" class="pl-4 text-4xl h-12 border border-grey" v-model="searchInput" @keyup.enter="prepareUrl(false, searchInput)">
      <p v-if="error">Please search for something...</p>
      <div class="mb-8">
        <button class="bg-green text-white font-bold py-2 px-4 my-8 rounded w-32 hover:bg-green-dark shadow-md" @click="prepareUrl(false, searchInput)">Search!</button>
        <div>
          <label for="dropdown">Amount of gifs to display:</label>
          <select class="border border-grey-dark"  id="dropdown" v-model.number="amountToShow" @change="changeAmount()">
            <option value="3">3</option>
            <option value="9" selected>9</option>
            <option value="30">30</option>
          </select>
          <button class="bg-blue text-white font-bold py-2 px-4 rounded" @click="prepareUrl(true, '')">Random</button>
          <button class="bg-red text-white font-bold py-2 px-4 rounded" @click="clear()">Clear</button>
        </div>
      </div>
      <p v-if="isLoading">Loading... </p>
      <div class="fixed z-50 pin-t pin-l w-full h-full  table singlegif-mask" v-if="gifClicked" @click="closeGif()">
      <!-- <div class="singlegif-mask" v-if="gifClicked" @click="closeGif()"> -->
        <div class="table-cell align-middle ">
          <img class="singlegif-image" :src="gifClickedUrl" alt="">
        </div>
      </div>
      <div class="gif-container">
        <img class="mx-auto my-0" v-if="isRandom" :src="randomGifUrl" @click="openGif(randomGifUrl)" alt="">
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
      error: false,
      gifClicked: false,
      isRandom: false,
      isLoading: false,
      amountToShow: 9
    };
  },

  methods: {
    prepareUrl(random, searchTerm) {
      if (searchTerm === "" && !random) {
        this.error = true;
        return;
      }
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
      this.error = false;
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
.gif-container {
  display: grid;
  grid-gap: 15px;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  .searched-img {
    height: 400px;
    width: 100%;
    object-fit: cover;
  }
  max-width: 1280px;
  margin: 0 auto;
}
.singlegif-mask {
  // position: fixed;
  // z-index: 9998;
  // top: 0;
  // left: 0;
  // width: 100%;
  // height: 100%;
  // background-color: rgba(0, 0, 0, 0.8);
  // display: table;
  background-color: rgba(0, 0, 0, 0.8);
  // transition: opacity 0.3s ease;
}

.singlegif-image {
  min-width: 800px;
}
</style>


