<template>
  <div
    class="home"
    v-loading="loading"
    element-loading-spinner="el-icon-loading"
    element-loading-background="rgba(0, 0, 0, 1)"
  >
    <Preview v-bind:id="id_film" v-bind:top="top"></Preview>
    <el-alert v-if="error != null" :title="error" type="error" class="mb-4"></el-alert>
    <el-row :gutter="10">
      <el-col :md="3">
        <div class="block sort">
          <el-cascader
            v-if="this.$session.get('lang') == 'fr'"
            class="input-clean"
            placeholder="Trier par"
            :options="sort"
            :props="props"
            v-model="valuesortby"
            @change="search"
            clearable
          ></el-cascader>
          <el-cascader
            v-else
            class="input-clean"
            placeholder="Sort by"
            :options="sort"
            :props="props"
            v-model="valuesortby"
            @change="search"
            clearable
          ></el-cascader>
        </div>
      </el-col>

      <el-col :md="3">
        <div class="block genders">
          <el-cascader
            v-if="this.$session.get('lang') == 'fr'"
            class="input-clean"
            placeholder="Genre"
            :options="genders"
            :props="props"
            v-model="valuegender"
            @change="search"
            clearable
          ></el-cascader>
          <el-cascader
            v-else
            class="input-clean"
            placeholder="Gender"
            :options="genders"
            :props="props"
            v-model="valuegender"
            @change="search"
            clearable
          ></el-cascader>
        </div>
      </el-col>

      <div class="filter">
        <el-col :md="3">
          <el-input
            v-if="this.$session.get('lang') == 'fr'"
            class="input-clean"
            min="0"
            max="10"
            placeholder="Note min"
            v-model="min_rate"
            type="number"
            @keyup.enter.native="search()"
          ></el-input>
          <el-input
            v-else
            class="input-clean"
            min="0"
            max="10"
            placeholder="Min rate"
            v-model="min_rate"
            type="number"
            @keyup.enter.native="search()"
          ></el-input>
        </el-col>
        <el-col :md="3">
          <el-input
            v-if="this.$session.get('lang') == 'fr'"
            class="input-clean"
            min="0"
            max="10"
            placeholder="Note max"
            v-model="max_rate"
            type="number"
            @keyup.enter.native="search()"
          ></el-input>
          <el-input
            v-else
            class="input-clean"
            min="0"
            max="10"
            placeholder="Max rate"
            v-model="max_rate"
            type="number"
            @keyup.enter.native="search()"
          ></el-input>
        </el-col>
        <el-col :md="3">
          <el-date-picker
            v-if="this.$session.get('lang') == 'fr'"
            class="input-clean"
            v-model="min_year"
            type="year"
            v-on:input="search()"
            placeholder="Année min"
          ></el-date-picker>
          <el-date-picker
            v-else
            class="input-clean"
            v-model="min_year"
            type="year"
            v-on:input="search()"
            placeholder="Min year"
          ></el-date-picker>
        </el-col>
        <el-col :md="3">
          <el-date-picker
            v-if="this.$session.get('lang') == 'fr'"
            class="input-clean"
            v-model="max_year"
            type="year"
            v-on:input="search()"
            placeholder="Année max"
          ></el-date-picker>
          <el-date-picker
            v-else
            class="input-clean"
            v-model="max_year"
            type="year"
            v-on:input="search()"
            placeholder="Max year"
          ></el-date-picker>
        </el-col>
      </div>

      <el-col :md="6">
        <div class="demo-input-suffix search">
          <el-input
            v-if="this.$session.get('lang') == 'fr'"
            class="input-clean"
            placeholder="Recherche"
            prefix-icon="el-icon-search"
            v-model="searchcontent"
            @keyup.enter.native="search()"
          ></el-input>
          <el-input
            v-else
            class="input-clean"
            placeholder="Search"
            prefix-icon="el-icon-search"
            v-model="searchcontent"
            @keyup.enter.native="search()"
          ></el-input>
        </div>
      </el-col>
    </el-row>

    <div
      class="list"
      v-loading="loading_search"
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(0, 0, 0, 1)"
    >
      <div class="infinite-list">
        <el-row :gutter="10">
          <el-col
            :xs="12"
            :sm="6"
            :md="6"
            :lg="4"
            :xl="4"
            v-for="film in films"
            v-bind:key="film.imdb_code"
            class="infinite-list-item div_film"
            :id="'div_film_'+film.id"
          >
            <img
              class="miniature"
              :src="film.medium_cover_image"
              :id="'film_'+film.id"
              @click="preview(film.id)"
            />
            <span class="title">{{film.title_english}}</span>
            <br />
            <small class="date">{{film.year}}</small>

            <small v-if="film.runtime" class="runtime">{{film.runtime}} min</small>
            <small v-if="film.director" class="director">Director : {{film.director}}</small>
            <br />

            <small class="note">
              <font-awesome-icon icon="star" />
              {{film.rating}}
            </small>
            <hr class="seen" v-for="movie in movies" v-if="movie.movieID == film.id" />
          </el-col>
        </el-row>
      </div>
      <div
          v-loading="loading_next"
          element-loading-spinner="el-icon-loading"
          element-loading-background="rgba(0, 0, 0, 1)"></div>
      </div>
  </div>
</template>


<script>
import Preview from "@/components/Preview.vue";

export default {
  name: "Home",
  mounted() {
    localStorage.setItem("ready", false);
    // console.log(localStorage.getItem("streamVideo"));
    if (localStorage.getItem("streamVideo") == "yes") {
      localStorage.setItem("streamVideo", "no");
      location.reload();
    }
    if (!this.$session.exists()) {
      this.$router.push("/login");
    } else {
      if (localStorage.getItem("video") == "yes") {
        localStorage.setItem("video", "no");
        location.reload();
      }
      this.scroll(this.person);
      this.load();
      this.getMovies();
    }
  },
  data() {
    return {
      props: { multiple: false },
      loading: true,
      loading_search: false,
      loading_next: false,
      sort: [
        {
          value: "like_count&order_by=desc",
          label: "Popularity +"
        },
        {
          value: "like_count&order_by=asc",
          label: "Popularity -"
        },
        {
          value: "title&order_by=asc",
          label: "A-Z"
        },
        {
          value: "title&order_by=desc",
          label: "Z-A"
        },
        {
          value: "year&order_by=asc",
          label: "Production year -"
        },
        {
          value: "year&order_by=desc",
          label: "Production year +"
        },
        {
          value: "rating&order_by=asc",
          label: "Rating -"
        },
        {
          value: "rating&order_by=desc",
          label: "Rating +"
        }
      ],
      genders: [
        {
          value: "Comedy",
          label: "Comedy"
        },
        {
          value: "Sci-fi",
          label: "Sci-fi"
        },
        {
          value: "Horror",
          label: "Horror"
        },
        {
          value: "Romance",
          label: "Romance"
        },
        {
          value: "Action",
          label: "Action"
        },
        {
          value: "Thriller",
          label: "Thriller"
        },
        {
          value: "Drama",
          label: "Drama"
        },
        {
          value: "Mystery",
          label: "Mystery"
        },
        {
          value: "Crime",
          label: "Crime"
        },
        {
          value: "Animation",
          label: "Animation"
        },
        {
          value: "Adventure",
          label: "Adventure"
        },
        {
          value: "Fantasy",
          label: "Fantasy"
        }
      ],
      page: 0,
      id_film: null,
      top: null,
      films: [],
      searchcontent: "",
      valuesortby: "",
      valuegender: "",
      pagesearch: 0,
      min_rate: null,
      max_rate: null,
      min_year: null,
      max_year: null,
      search_min_year: null,
      search_max_year: null,
      error: null,
      request: "limit=20&page=" + this.page,
      movies: []
    };
  },
  methods: {
    cleanStr(str) {
      if (str) {
        let s2 = "";
        let alphanum = new RegExp("^[a-zA-Z0-9_]*$");
        for (var i = 0; i < str.length; i++) {
          if (alphanum.test(str[i])) s2 += str[i];
        }
        return s2;
      }
    },
    cleanNumber(str) {
      if (str) {
        let s2 = "";
        let alphanum = new RegExp("^[0-9_]*$");
        for (var i = 0; i < str.length; i++) {
        if (alphanum.test(str[i])) s2 += str[i];
        }
        return s2;
      }
    },
    getMovies() {
      this.axios
        .get(
          "https://localhost:5001/api/v1/users/" +
            encodeURI(this.$session.get("username")) +
            "/movies",
          {
            headers: {
              access_token: localStorage.getItem("token")
            }
          }
        )
        .catch(err =>{
          // console.log(err)
        })
        .then(response => {
          for (var i = 0; i < response.data.length; i++) {
            this.movies.push(response.data[i]);
          }
        })
        
    },
    offset(el) {
      var rect = el.getBoundingClientRect(),
        scrollLeft = window.pageXOffset || document.documentElement.scrollLeft,
        scrollTop = window.pageYOffset || document.documentElement.scrollTop;
      return { top: rect.top + scrollTop, left: rect.left + scrollLeft };
    },
    preview(id) {
      var link = document.getElementById("div_film_" + id);
      var offset = this.offset(link);
      var top = offset.top;
      var bottom_div = top + link.clientHeight;
      var bottom = bottom_div;

      var divs = document.getElementsByClassName("div_film");

      for (var i = 0; i < divs.length; i++) {
        link = divs[i];
        offset = this.offset(link);
        top = offset.top;
        bottom_div = top + link.clientHeight;
        if (bottom_div == bottom) divs[i].style.marginBottom = "500px";
        else divs[i].style.marginBottom = "30px";
      }

      link = document.getElementById("div_film_" + id);
      offset = this.offset(link);
      top = offset.top;
      bottom_div = top + link.clientHeight;
      this.top = bottom_div;
      this.id_film = id;
      document
        .getElementById("film_" + id)
        .scrollIntoView({ behavior: "smooth" });
    },
    load() {
      this.loading = true;
      this.page++;
      this.axios
        .get(
          "https://localhost:5001/api/v1/yts/sort_by=like_count&limit=20/page=" +
            this.page +
            "/" +
            this.min_rate +
            "/" +
           this.max_rate +
            "/" +
            this.search_min_year +
            "/" +
            this.search_max_year
        )
        .catch(err =>{
          // console.log(err)
        })
        .then(response => {
          this.films = response.data;
          this.loading = false;
        });
    },
    search() {
      this.loading_search = true;
      this.error = null;
      this.page = 1;
      this.id_film = null;
      this.request = "limit=20";
      if (this.valuesortby[0] == undefined && this.searchcontent == "")
        this.request += "&sort_by=like_count";
      else if (this.valuesortby[0] == undefined && this.searchcontent != "")
        this.request += "&sort_by=title&order_by=asc";
      else {
        this.request += "&sort_by=" + this.valuesortby[0];
      }
      if (this.valuegender[0] != undefined)
        this.request += "&genre=" + this.valuegender[0];
      if (this.searchcontent != "")
        this.request += "&query_term=" + this.cleanStr(this.searchcontent);

      if (this.min_rate == "") this.min_rate = null;
      if (this.max_rate == "") this.max_rate = null;
      if (this.min_year == "") this.min_year = null;
      if (this.max_year == "") this.max_year = null;

      if (
        this.min_rate > this.max_rate &&
        this.max_rate != null &&
        this.min_rate > this.max_year
      ) {
        this.min_rate = null;
        this.max_rate = null;
        this.error = "Careful, max rate can't be lower than min rate";
      }
      if (
        this.min_year != null &&
        new Date(this.min_year).getFullYear() <= new Date().getFullYear()
      )
        this.search_min_year = new Date(this.min_year).getFullYear();
      else this.search_min_year = null;
      if (
        this.max_year != null &&
        new Date(this.max_year).getFullYear() <= new Date().getFullYear()
      )
        this.search_max_year = new Date(this.max_year).getFullYear();
      else this.search_max_year = null;

      if (
        this.min_year != null &&
        this.max_year != null &&
        this.min_year > this.max_year
      ) {
        this.min_year = null;
        this.max_year = null;
        this.search_min_year = null;
        this.search_max_year = null;
        this.error = "Careful, max year can't be lower than min year";
      }


      if(this.min_rate != null)
        this.min_rate = this.cleanNumber(this.min_rate)
      if(this.max_rate != null)
        this.max_rate = this.cleanNumber(this.max_rate)
    
  
      this.axios
        .get(
          "https://localhost:5001/api/v1/yts/" +
            this.request +
            "/page=" +
            this.page +
            "/" +
            this.min_rate +
            "/" +
            this.max_rate +
            "/" +
            this.search_min_year +
            "/" +
            this.search_max_year
        )
        .catch(err =>{
          // console.log(err)
        })
        .then(response => {
          this.films = response.data;
          this.loading_search = false;
          if (this.films.length < 1) {
            if (
              this.min_rate != null &&
              this.max_rate != null &&
              this.search_min_year != null &&
              this.search_max_year != null
            ) {
              this.next();
            }
          }
        });
    },
    async next() {
      var response = [];
      while (this.films.length < 1) {
        this.page++;
        response = await this.axios.get(
          "https://localhost:5001/api/v1/yts/" +
            this.request +
            "/page=" +
            this.page +
            "/" +
            this.min_rate +
            "/" +
            this.max_rate +
            "/" +
            this.search_min_year +
            "/" +
            this.search_max_year
        )
        .catch(err =>{
          // console.log(err)
        })
        for (var i = 0; i < response.data.length; i++) {
          this.films.push(response.data[i]);
        }
        if (this.films.length < 1) 
          this.next();
      }
    },
    scroll() {
      window.onscroll = () => {
        let bottomOfWindow =
          Math.ceil(document.documentElement.scrollTop + window.innerHeight) ===
          document.documentElement.offsetHeight;

        if (bottomOfWindow) {
          this.loading_next = true;
          this.page++;
          this.request = "limit=20&page=" + this.page;
          if (this.valuesortby[0] == undefined)
            this.request += "&sort_by=like_count";
          else {
            this.request += "&sort_by=" + this.valuesortby[0];
          }
          if (this.valuegender[0] != undefined)
            this.request += "&genre=" + this.valuegender[0];
          if (this.searchcontent != "")
            this.request += "&query_term=" + this.cleanStr(this.searchcontent);

          this.axios
            .get(
              "https://localhost:5001/api/v1/yts/" +
                this.request +
                "/page=" +
                this.page +
                "/" +
                this.min_rate +
                "/" +
                this.max_rate +
                "/" +
                this.search_min_year +
                "/" +
                this.search_max_year
            )
            .catch(err =>{
              // console.log(err)
            })
            .then(response => {
              for (var i = 0; i < response.data.length; i++) {
                this.films.push(response.data[i]);
              }
              this.loading_next = false;
            });
        }
      };
    }
  },
  components: {
    Preview
  }
};
</script>
