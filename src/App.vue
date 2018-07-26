<template>
  <div id="app">
    <a href="https://github.com/quelhasu/movie-income-guesser" target="blank_" class="github-corner" aria-label="View source on Github">
  <svg id="github" width="80" height="80" viewBox="0 0 250 250" style="fill:#fff; position: absolute; top: 0; border: 0; right: 0;"
    aria-hidden="true">
    <path d="M0,0 L115,115 L130,115 L142,142 L250,250 L250,0 Z"></path>
    <path d="M128.3,109.0 C113.8,99.7 119.0,89.6 119.0,89.6 C122.0,82.7 120.5,78.6 120.5,78.6 C119.2,72.0 123.4,76.3 123.4,76.3 C127.3,80.9 125.5,87.3 125.5,87.3 C122.9,97.6 130.6,101.9 134.4,103.2"
      fill="currentColor" style="transform-origin: 130px 106px;" class="octo-arm"></path>
    <path d="M115.0,115.0 C114.9,115.1 118.7,116.5 119.8,115.4 L133.7,101.6 C136.9,99.2 139.9,98.4 142.2,98.6 C133.8,88.0 127.5,74.4 143.8,58.0 C148.5,53.4 154.0,51.2 159.7,51.0 C160.3,49.4 163.2,43.6 171.4,40.1 C171.4,40.1 176.1,42.5 178.8,56.2 C183.1,58.6 187.2,61.8 190.9,65.4 C194.5,69.0 197.7,73.2 200.1,77.6 C213.8,80.2 216.3,84.9 216.3,84.9 C212.7,93.1 206.9,96.0 205.4,96.6 C205.1,102.4 203.0,107.8 198.3,112.5 C181.9,128.9 168.3,122.5 157.7,114.1 C157.9,116.9 156.7,120.9 152.7,124.9 L141.0,136.5 C139.8,137.7 141.6,141.9 141.8,141.8 Z"
      fill="currentColor" class="octo-body"></path>
  </svg>
</a>
    <div class="header">
      <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.0.13/css/all.css" integrity="sha384-DNOHZ68U8hZfKXOrtjWvjxusGo9WQnrNx2sqG0tfsghAvtVlRW3tvkXWZh58N9jp" crossorigin="anonymous">
      <!-- <h1>WHO'S MAKING THE <span id="mainger">HIGHEST</span> INCOME ?</h1> -->
      <vue-typer 
      :text="typerText" 
      :repeat="typerRepeat" 
      :shuffle='true'
      :pre-type-delay='1500' 
      :type-delay='50' 
      :pre-erase-delay='5000'
      :erase-delay='75' 
      :erase-on-complete="typerErase" 
      erase-style='select-back' 
      caret-animation='smooth'
    ></vue-typer>

    </div>
    <div class="cards">
      <div v-for="(card, index) in cards" class="card" :ref="'card'+index" @click="verify('card'+index, index)" :key="index">
        <div v-show="verified" :ref="'overlay'+index" class="card-overlay">
          <ICountUp :ref="'counter'+index" :startVal=0 :end-val=0 :duration="waitingTime" :options='{useEasing: true, useGrouping: true, separator: ",",decimal: ".",prefix: "$", suffix: ""}' />
        </div>
        <img class="card-image" :ref="'img'+index" :src="'http://image.tmdb.org/t/p/w400/' + card.movie.poster_path" @error="defaultImage('img'+index)">
        <div class="card-divider">
          <div class="vote">
            <i class="fas fa-film" :ref="'icon'+index"/>
        </div> 
        </div>
        <div class="card-body">
          <h2 class="body-title">{{card.movie.title}}</h2>
          <div class="body-inline">
            <p class="body-inline-title">Actors</p>
            <p class="body-inline-text" v-for="(actor, index2) in card.actors" :key="index2">
              {{actor.name}},
            </p>
          </div>
          <div class="body-inline">
            <p class="body-inline-title">Director</p>
            <p class="body-inline-text">{{card.director.name}}</p>
          </div>
        </div>
        <div class="card-footer">
          <div class="card-footer-left">
            <i class="fas fa-calendar"/>
            {{card.movie.release_date | yearDate}}
          </div>
          <div style="display:flex">
            <p class="m-r-sm" style="font-size:10px" v-for="genre in card.movie.genres">
              {{genre.name}} 
            </p>
          </div>
          <div class="card-footer-right">
            {{card.movie.vote_average}}
            <i class="fas fa-star"/>
          </div>
        </div>
      </div>
    </div>
    <div v-show="wins > 0||looses > 0" class="stats">
      <div class="win" :style="{width: (wins/(wins+looses)*100) + '%'}">{{wins}}</div>
      <div class="loose" :style="{width: (looses/(wins+looses)*100) + '%'}">{{looses}}</div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import ICountUp from "vue-countup-v2";
import moment from "moment";
import VueTyperPlugin from "vue-typer";
import Vue from "vue";

Vue.use(VueTyperPlugin);

export default {
  name: "app",
  components: {
    ICountUp
  },
  data: function() {
    return {
      cards: [
        {
          movie: {},
          actors: ["unknown"],
          director: ""
        },
        {
          movie: {},
          actors: ["unknown"],
          director: ""
        }
      ],
      movies: [],
      errors: [],
      firstC: {},
      secondC: {},
      verified: false,
      waitingTime: 2,
      most: "",
      winner: "",
      wins: 0,
      looses: 0,
      typerText: ["WHO'S MAKING THE HIGHEST INCOME?"],
      typerArray: [
        "Really?",
        "Oh, you're still here?",
        "How are you?",
        "Not great movies, huh..?",
        "???",
        "Don't you get pissed off?",
        "So?",
        "I'll be back, just a minute..",
        "You're not very good..",
        "Oh you can check my GitHub"
      ],
      typerErase: false,
      typerRepeat: "0"
    };
  },
  filters: {
    yearDate: function(value) {
      if (value) return moment(String(value)).format("YYYY");
    }
  },
  methods: {
    async getMoviesPage(page) {
      await axios
        .get(
          "https://api.themoviedb.org/3/movie/top_rated?api_key=40e3eb17197b2edad779e85fceb5797f&language=en-US&page=" +
            page
        )
        .then(response => {
          this.movies = response.data.results;
        })
        .catch(e => {
          this.errors.push(e);
        });
    },
    async getMovieCredits(index) {
      await axios
        .get(
          "https://api.themoviedb.org/3/movie/" +
            this.cards[index].movie.id +
            "/credits?api_key=40e3eb17197b2edad779e85fceb5797f"
        )
        .then(response => {
          this.cards[index].actors = response.data.cast.slice(0, 4);
          this.cards[index].director = response.data.crew[0];
        });
    },
    async getMovieDetails(index, array_index) {
      var movie = {};
      await axios
        .get(
          "https://api.themoviedb.org/3/movie/" +
            this.movies[array_index].id +
            "?language=en-US&api_key=40e3eb17197b2edad779e85fceb5797f"
        )
        .then(response => {
          movie = response.data;
          this.cards[index].movie = movie;
        })
        .catch(e => {
          this.errors.push(e);
        });
    },
    changeTypeText() {
      var newText = this.typerArray[
        Math.floor(Math.random() * this.typerArray.length)
      ];
      this.typerText = this.typerArray;
      this.typerErase = true;
      this.typerRepeat = "Infinity";
    },
    defaultImage(img) {
      this.$refs[img][0].src =
        "https://celebritybucks.com/images/celebs/full/default.gif";
    },
    init() {
      if (this.verified) {
        this.$refs["overlay" + this.winner][0].classList.remove("card-win");
        this.$refs["overlay" + (this.winner + 1) % 2][0].classList.remove(
          "card-loose"
        );
      }
      this.verified = false;
      [0, 1].forEach(index => {
        this.$refs["icon" + index][0].classList.remove("fa-check", "fa-times");
        this.$refs["icon" + index][0].classList.add("fa-film");
        this.$refs["counter" + index][0].update(0);
        this.$refs["card" + index][0].classList.remove(
          "card-selected",
          "card-not-selected"
        );
        index == 0
          ? this.$refs["card" + index][0].classList.add(
              "animated",
              "bounce-left"
            )
          : this.$refs["card" + index][0].classList.add(
              "animated",
              "bounce-right"
            );

        // if (index == 0)
        //   this.$refs["card" + index][0].classList.add(
        //     "animated",
        //     "bounceInLeft"
        //   );
        // else
        //   this.$refs["card" + index][0].classList.add(
        //     "animated",
        //     "bounceInRight"
        //   );
      });
    },
    async selectMovies() {
      this.init();
      var pages = 15;
      var page = 1;
      var array_index = 0;
      do {
        -await Promise.all(
          [0, 1].map(async index => {
            do {
              page = Math.floor(Math.random() * (pages - 1) + 1);
              await this.getMoviesPage(page);
              array_index = Math.floor(Math.random() * this.movies.length);
              await this.getMovieDetails(index, array_index);
            } while (
              this.cards[index].movie.revenue == 0 ||
              this.cards[index].movie === null ||
              this.cards[index].movie.popularity < 10
            );
          })
        );
      } while (this.cards[0].movie.id == this.cards[1].movie.id);
      await Promise.all(
        [0, 1].map(async index => {
          await this.getMovieCredits(index);
        })
      );
    },
    verify: function(card, index_card) {
      var vm = this;
      if (!this.verified) {
        this.verified = true;
        [0, 1].forEach(index => {
          this.$refs["counter" + index][0].update(
            this.cards[index].movie.revenue
          );
          this.$refs["card" + index][0].classList.remove(
            "animated",
            "bounce-left",
            "bounce-right"
          );
          this.$refs["card" + (index_card + 1) % 2][0].classList.add(
            "card-not-selected"
          );
        });

        this.$refs[card][0].classList.add("card-selected");
        this.cards[0].movie.revenue > this.cards[1].movie.revenue
          ? (this.winner = 0)
          : (this.winner = 1);

        setTimeout(function() {
          vm.updateCard(vm);
        }, (this.waitingTime + 1) * 1000);
        setTimeout(function() {
          if (!this.typerErase) vm.changeTypeText();
        }, (this.waitingTime + 2) * 1000);
        setTimeout(function() {
          vm.selectMovies();
          card === "card" + vm.winner ? vm.wins++ : vm.looses++;
        }, (this.waitingTime + 3) * 1000);
      }
    },
    updateCard: function(vm) {
      // Cards
      vm.$refs["card" + vm.winner][0].classList.remove("card-not-selected");
      vm.$refs["card" + vm.winner][0].classList.add("card-selected");
      vm.$refs["card" + (vm.winner + 1) % 2][0].classList.remove(
        "card-selected"
      );
      vm.$refs["card" + (vm.winner + 1) % 2][0].classList.add(
        "card-not-selected"
      );

      // Change icon winner card
      vm.$refs["overlay" + vm.winner][0].classList.add("card-win");
      vm.$refs["icon" + vm.winner][0].classList.remove("fa-film");
      vm.$refs["icon" + vm.winner][0].classList.add("fa-check");

      // Change icon looser card
      vm.$refs["overlay" + (vm.winner + 1) % 2][0].classList.add("card-loose");
      vm.$refs["icon" + (vm.winner + 1) % 2][0].classList.remove("fa-film");
      vm.$refs["icon" + (vm.winner + 1) % 2][0].classList.add("fa-times");
    }
  },
  mounted: function() {
    this.selectMovies();
    // axios
    //   .get("https://celebritybucks.com/developers/export/JSON")
    //   .then(response => {
    //     this.celebrities = response.data.CelebrityValues;
    //     this.selectCelebrities();
    //   })
    //   .catch(e => {
    //     this.errors.push(e);
    //   });
  }
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
}
#app {
  font-family: "Roboto", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* overflow: hidden; */
}
</style>

<style lang="scss">
@import "./assets/style.sass";
@import "./assets/spacing.scss";
</style>
