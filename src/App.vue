<template>
  <div id="app" class="font-sans text-center antialiased text-blue-darkest mt-8">
    <div class="container mb-8">

      <img src="./assets/giphy.gif">
      <h1 class="font-semibold my-6">{{ msg }}</h1>
      <input type="text" class="pl-4 text-2xl h-12 border border-grey" v-model="searchInput" @keyup.enter="prepareUrl(searchInput)">
      <p v-if="error">Please search for something...</p>
      <div class="mb-8">
        <button class="bg-green text-white font-bold py-2 px-4 my-8 rounded w-32 hover:bg-green-dark shadow-md" @click="prepareUrl(searchInput)">Search!</button>
        <div class="container">
          <label for="dropdown">Amount of gifs to display:</label>
          <select class="border border-grey-dark"  id="dropdown" v-model.number="amountToShow" @change="changeAmount()">
            <option v-for="option in options" :key="option.id">
              {{ option }}
            </option>
          </select>
          <button class="text-blue font-bold py-2 px-2 rounded" @click="prepareRandomUrl(searchInput)">Random</button>
          <button class="text-grey-dark font-bold py-2 px-2 rounded" @click="clear()">Clear</button>
        </div>
      </div>
      <p v-if="isLoading">Loading... </p>
      <div class="fixed z-50 pin-t pin-l w-full h-full  table singlegif-mask" v-if="gifIsClicked" @click="closeGif()">
        <div class="table-cell align-middle ">
          <img class="singlegif-image" :src="gifClicked.url" alt="">
        </div>
      </div>
      <div class="gif-container">
        <img class="mx-auto my-0" v-if="isRandom" :src="randomGif.url" @click="openGif(randomGif)" alt="">
        <img class="searched-img" v-for="gif in gifs" v-if="gif.isVisible" :src="gif.url" alt="" :key="gif.id" @click="openGif(gif)">
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
      gifs: [],
      gifClicked: "",
      error: false,
      gifIsClicked: false,
      isRandom: false,
      isLoading: false,
      amountToShow: 3,
      options: ["3", "9", "30"],
      randomGif: {}
    };
  },

  methods: {
    prepareRandomUrl(searchTerm) {
      this.isRandom = true;
      let url = `${this.randomUrl + this.apiKey}`;
      this.fetchGifs(url);
    },

    prepareUrl(searchTerm) {
      this.isRandom = false;
      if (searchTerm === "" && !this.isRandom) {
        this.error = true;
        return;
      }
      let limit = 30;
      let url = `${this.searchUrl +
        this.apiKey}&q=${searchTerm}&limit=${limit}`;
      this.fetchGifs(url);
      this.searchInput = "";
      this.error = false;
    },

    fetchGifs(url) {
      this.toggleLoading();
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
        this.randomGif = {
          url: `https://media.giphy.com/media/${json.data.id}/giphy.gif`,
          isVisible: true
        };
      } else {
        // We fetch multiple gifs so need to iterate over them
        this.gifs = json.data.map(gif => gif.id).map(gifId => {
          let gif = {
            url: `https://media.giphy.com/media/${gifId}/giphy.gif`,
            isVisible: true
          };
          return gif;
        });
        this.changeAmount();
      }
    },

    toggleLoading() {
      this.isLoading = !this.isLoading;
    },

    clear() {
      this.gifs = [];
      this.randomGif = "";
    },

    changeAmount() {
      // this.gifs = [];
      console.log("called");
      if (this.isRandom) {
        this.amountToShow = 1;
        return;
      }
      this.gifs.map((gif, index) => {
        if (index < this.amountToShow) gif.isVisible = true;
        else gif.isVisible = false;
      });
    },

    openGif(gif) {
      this.gifClicked = gif;
      this.gifIsClicked = true;
      document.body.style.overflowY = "hidden";
    },

    closeGif() {
      this.gifClicked = null;
      this.gifIsClicked = false;
      document.body.style.overflowY = "auto";
    }
  },
  mounted() {
    document.addEventListener("keydown", e => {
      if (this.gifIsClicked && e.keyCode == 27) {
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
  max-width: 90vw;
}
</style>


