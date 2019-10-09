<template>
<div class="flex flex-col ">
  <div id="welcome" class="p-4 bg-gray-800 text-center">
    <h2 class="text-white">We found You in</h2>
    <h1 class="text-indigo-200 text-3xl">{{location}}</h1>
    <h3 class="text-xs"><a @click.prevent="promptForZip" class="underline text-white" href="#">Not Here?</a></h3>
  </div>
  <div class="flex-grow mt-12 flex flex-col sm:w-full md:w-2/3 mx-auto" id="weather">
    <h2 class="text-3xl text-indigo-400 text-center">{{today}}</h2>
    <div class="mt-16">
      <div class="text-2xl text-gray-800 text-center" v-if="!weatherPeriods.length">
        <p v-if="error">{{error}}</p>
        <p v-else>Waiting on the Weather!</p>
      </div>
      <div  v-if="weatherPeriods.length"
            class="flex items-center justify-center text-center w-full text-gray-600">
        <div class="" v-for="period in weatherPeriods">
          <div class="border-t border-b text-sm text-gray-800 sm:px-2 md:px-4 lg:px-8 py-2">
            <p>{{formatWeatherPeriod(period.dt)}}</p>
          </div>
          <div class="pl-1 mt-1 text-gray-500 text-lg">
            <p>{{formatTemp(period.main.temp)}}&deg;</p>
          </div>
          <div class="w-8 text-center inline-block">
            <img v-if="true" class="" :src="buildIconUrl(period.weather)">
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="bg-gray-800">
    <form @submit.prevent="fetchByZip" class="p-6 pb-12 mx-auto w-64 flex flex-col">
      <label class="text-white text-bold text-2xl" for="zip">Enter your zip code</label>
      <input class="mt-2 inline-block shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" type="text" v-model="zip" ref="zip" required="required" pattern="[0-9]{5}" placeholder="Zip Code">
      <input type="submit" class="mt-2 px-2 py-1 rounded inline-block text-white bg-indigo-400 w-100" value="Submit">
    </form>
  </div>
</div>
</template>
<script>
  /* eslint-disable */
import axios from 'axios';
import moment from 'moment';
import WeatherIcon from "./WeatherIcon.vue"
const envVars = require('../env.js')

export default {
    // components: {
    //     WeatherIcon
    // },
    mounted() {
        this.setLatLong()
    },
    data() {
        return {
            error: null,
            zip: null,
            lat: null,
            lon: null,
            weatherLocation: null,
            weatherPeriods: [],
        }
    },
    methods: {
        fetchByZip() {
            if (!this.zip || !this.zip.match(/^\d{5}$/)) {
                alert('Please enter an valid 5 digit zipcode.')
                return;
            }
            this.weatherLocation = null;
            this.weatherPeriods = [];
            this.fetchWeather(null, this.zip)
        },
        setLatLong() {
            navigator
                .geolocation
                .getCurrentPosition(this.fetchWeather, this.promptForZip)
        },
        promptForZip() {
            this.$refs.zip.value = ''
            this.$refs.zip.focus()
            alert("Enter a zip code, or enable geolocation.")
        },
        fetchWeather(position, zip = null) {
            const url = this.buildAPIUrl(position, zip)

            if (url === null) {
                return this.promptForZip()
            }
            fetch(url)
                .then(response => {
                    response.json()
                        .then(data => this.parseWeatherData(data))
                        .catch(err => {
                            alert('Unable to parse weather data')
                            console.log('err',err)
                            this.error = err.message
                        })
                })
                .catch(err => {
                    alert('Unable to fetch weather data');
                    this.error = err.message
                    console.log(err.code);
                });
            return;

            axios
                .get(url)
                .then(resp => {
                    this.parseWeatherData(resp.data)
                })
                .catch(err => {
                    alert('We are unable to fetch the weather for you.')
                    console.log(err)
                })
        },
        parseWeatherData(data) {
            console.log(data)
            if (data.cod && data.cod !== "200") {
                alert('Pares says We are unable to fetch weather.');
                this.error = data.message
                return
            }

            this.weatherLocation = data.city.name
            this.weatherPeriods = data.list
        },
        buildAPIUrl(position, zip = null) {
            if (zip === null) {
                return `http://api.openweathermap.org/data/2.5/forecast?units=imperial&appid=${envVars.apiKey}&lat=${position.coords.latitude}&lon=${position.coords.longitude}&cnt=8`
            }
            if (zip) {
                return `http://api.openweathermap.org/data/2.5/forecast?units=imperial&appid=${envVars.apiKey}&zip=${this.zip},us&cnt=8`
            }
            return null;
        },
        formatWeatherPeriod(value) {
            return moment.unix(value).format("ha").replace('m','')
        },
        formatTemp(temp) {
            return Math.floor(temp)
        },
        buildIconUrl(weather) {
            if (!weather.length) {
                return 'http://openweathermap.org/img/wn/10d@2x.png'
            }

            const icon = weather[0].icon//.replace('n', 'd')
            return `http://openweathermap.org/img/wn/${icon}@2x.png`
        }
    },
    computed: {
        today() {
            return moment().format('MMMM Do, YYYY')
        },
        location() {
            const here =  this.weatherLocation || this.zip || 'Still Thinking ...'
            return here
        }
    }
}
</script>
