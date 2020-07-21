<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md"> <!-- col padding top large -->
      <q-input 
        v-model="search" 
        @keyup.enter="getWeatherBySearch"
        placeholder="Rechercher"
        dark
        borderless
      >
        <template v-slot:before> <!--onclick get Location -->
          <q-icon 
            @click="getLocation" 
            name="my_location"
            />
        </template>
        <template v-slot:append>
          <q-btn 
            @click="getWeatherBySearch"
            round dense 
            flat 
            icon="search" 
          />
        </template>
      </q-input>
    </div>  

<!-------------------------------------------------------------------- -->

<!-- Les templates regroupent les éléments et les if else -->
    <template v-if="weatherData"> <!-- Si on a des données dans WeatherData -->
      <div class="col text-white text-center"> <!-- affichage nom de ville et la température en text -->
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degres">&deg;C</span>
        </div>
      </div>
      
      <div class="col text-center">
        <img :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`">
      </div>      
    </template>
    <template v-else>
      <div class="col column text-center text-white"> <!-- si weatherData = null afficher "Quasar Wheather" -->
        <div class="col text-h2 text-weight-thin">
          Quasar<br>Wheather
        </div>
          <!-- q-bouton contient le bouton pour chercher la location -->
          <q-btn
          @click="getLocation"
          class="col" 
          flat
          >
          <q-icon left size="3em" name="my_location" /> <!-- icon rechercher avec sa localisation -->
          <div>Trouver ma position</div>
          </q-btn>
      </div>
    </template>

<!--------------------------------------------------------------------------------------- -->

    <div class="col skyline"></div> <!-- fond ville png -->

  </q-page>
</template>

<!-- contient les script "search", "wheaterData", la methode getLocation ... -->
<script>
export default {
  name: 'PageIndex',
  data() {
    return {
      search: '',
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: 'https://api.openweathermap.org/data/2.5/weather?',
      apiKey: '*************************',
    }
  },

// Methode de la class bgClass utilisé sur le q-page 
// Si weatherData, Si l'icon dans 3weatherData>weather>array0>icon3 termine par 'n' 
// alors mettre la class bg-night, sinon mettre la class bg-day.
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        }
        else {
          return 'bg-day'
        }
      }
    }
  },
// method pour la position actuelle
  methods: {
    getLocation() {
      this.$q.loading.show() // affichage du Loader 

// si la plateforme est electron, get cette adresse de geolocalisation
      if (this.$q.platform.is.electron) {
          this.$axios.get('https://freegeoip.app/json/').then(response => { 
          this.lat = response.data.latitude
          this.lon = response.data.longitude
          this.getWeatherByCoords()
        })
      }
// sinon utiliser la geolocalisation
      else {
          navigator.geolocation.getCurrentPosition
        (position => {
          console.log('position: ', position)
          this.lat = position.coords.latitude
          this.lon = position.coords.longitude
          this.getWeatherByCoords()
        })
      }  

    },
// methode pour la localisation par coordonnées
    getWeatherByCoords() {
      this.$q.loading.show() // affichage du Loader 
      this.$axios(`${this.apiUrl}&lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`).then(response => {
        this.weatherData = response.data
        this.$q.loading.hide() // disparition loader
      })
    },
// méthode pour la localisation par recherche
    getWeatherBySearch() {
      this.$q.loading.show() // affichage du Loader 
      this.$axios(`${this.apiUrl}q=${this.search}&appid=${this.apiKey}&units=metric`).then(response => {
      this.weatherData = response.data
      this.$q.loading.hide()
      }) 
    },
  }
}
</script>

<!-- contient le style css avec Sass -->
<style lang="sass"> 
  .q-page
    background: linear-gradient(to bottom, #136a8a, #267871)
    &.bg-night
      background: linear-gradient(to bottom, #000000, #414345)
    &.bg-day
      background: linear-gradient(to bottom, #00b4db, #2F80ED)
  .degres
    top: -44px
  .skyline
    flex: 0 0 100px
    background: url(../statics/icons/skyline.png)
    background-size: contain
    background-position: center bottom
  
</style>
