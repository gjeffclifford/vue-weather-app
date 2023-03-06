<template>
  <q-page class="flex flex-center">
    <div class="q-pa-md container-weather">
      <div class="row">
        <div class="col">
          <q-select
            filled
            bottom-slots
            use-input
            @filter="filterFn"
            v-model="selectedWeather"
            :options="options"
            label="Search Place Here"
            @update:model-value="getWeather"
            clearable
          >
          </q-select>
        </div>
      </div>
      <div class="row">
        <div class="col" style="max-height: 70vh; overflow-y:auto;">
          <table style="width: 100%;">
            <tbody v-if="weathers.length">
              <tr v-for="(weather, index) in weathers" :key="index">
                <td>
                  <img :src=getImage([weather.weather[0].icon]) alt="">
                </td>
                <td>{{ toUcWords(weather.weather[0].description) }}</td>
                <td>{{ toCalendarTime(weather)}}</td>
                <td>{{ toCelsius(weather.main.temp) }}&deg;C</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import { ref, onMounted } from 'vue'
import moment from "moment";

export default defineComponent({
  name: 'IndexPage',
  data: () => {
    return {
      options: [],
      selectedWeather: "",
      searchDebounce: null,
      weathers: []
    }
  },
  watch: {
    selectedWeather(newValue){
      if(newValue !== null) {
        this.getWeather(newValue.value)
      }
    }
  },
  computed: {
    mapUrl() {
      let properties = this.selectedWeather.value.properties
      // return "google.com"
      return `https://maps.google.com/maps?q=${properties.lat},${properties.lon}&z=15&output=embed`
    }
  },
  methods: {
    async filterFn(keyword, update) {
      if(keyword) {
        clearTimeout(this.searchDebounce)
        this.searchDebounce = window.setTimeout(async () => {
          await this.$store.dispatch('weather/getPlaces', {keyword}).then(async (response) => {
            update(() => {
              this.options = response.features.map((opt) => {
                return {
                  label: opt.properties.formatted,
                  value: opt
                }
              })
            })
          })
        }, 1000)
      }
    },
    getWeather(value){
      if (value) {
        this.$store.dispatch('weather/getWeather', {lat: value.properties.lat, lon: value.properties.lon}).then(async (response) => {
          this.weathers = response.list
        })
      }
    },
    toCelsius(temperature) {
      return (temperature-273.15).toFixed(1)
    },
    toUcWords(str) {
      return str.toLowerCase().replace(/\b[a-z]/g, function(letter) {
          return letter.toUpperCase();
      });
    },
    getImage(icon) {
      return `http://openweathermap.org/img/wn/${icon}@2x.png`
    },
    toCalendarTime(weather) {
      return moment(weather.dt_txt).calendar()
    }
  },
  setup() {
    onMounted(() => {
      console.log(process.env.WEATHER_API)
    })
  }
})
</script>
