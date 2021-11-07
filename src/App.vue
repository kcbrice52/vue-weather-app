<template>
  <div id="app" :class="typeof weather.main != 'undefined'  && weather.main.temp > 65 ? 'warm' : 'cold'">
    <main>
      <div class="search-box">
        <input
          type="text"
          class="search-bar" 
          placeholder="Search..."
          v-model="query"
          @keypress="fetchWeather"
        />
      </div>
      <div class="weather-wrap" v-if="typeof weather.main != 'undefined'">
        <div class="location-box">
          <div class="date">{{ dateBuilder() }}</div>
        </div>
        <div class="weather-box">
          <div class="location">{{ weather.name }}, {{weather.sys.country }}</div>
          <div class="temp">{{ Math.round(weather.main.temp) }}°F</div>
          <br>
          <img v-bind:src="'http://openweathermap.org/img/wn/' + weather.weather[0].icon + '@2x.png'">
          <div class="weather">{{ weather.weather[0].main }}</div>
        </div>
        <div class="forecast-box">
          <template v-if="forecastView === 'hourly'">
          <button class="button floated-right" v-on:click="viewDailyForecast">View Daily</button>
          <h2>Hourly Forecast</h2>
          <br>
          <div class="row">
            <div class="column-2" v-for="hour in 24" :key="hour">
              <div class="hour-box">
                <div class="hour">{{ moment(hourlyForecast[hour-1].dt*1000).format('h A')}}</div>
                <div class="">{{ hourlyForecast[hour-1].temp }}&#176;F</div>
              </div>
            </div>
          </div>
          </template>
          <template v-if="forecastView === 'daily'">
          <button class="button floated-right" v-on:click="viewHourlyForecast">View Hourly</button>
          <h2>Daily Forecast</h2>
          <br>
          <div class="row">
            <div class="column" v-for="day in dailyForecast.list" :key="day.dt">
              <div class="day-box">
                <div class="day"> {{ moment(day.dt*1000).format('DD ddd') }} </div>
                <div class="temp">{{  Math.round(day.temp.day) }}&#176;F</div>
                <div class="temp">{{  Math.round(day.temp.night) }}&#176;F</div>
                <img v-bind:src="'http://openweathermap.org/img/wn/' + day.weather[0].icon + '@2x.png'">
                <div>{{ day.weather[0].main }}</div>
              </div>
            </div>
          </div>
          </template>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import moment from 'moment';

export default {
  name: 'App',
  data () {
    return {
      api_key: '2209bb742e00865106d7950710ecf8ca',
      url_base: 'https://api.openweathermap.org/data/2.5/',
      query: '',
      weather: {},
      dailyForecast: {},
      hourlyForecast: {},
      forecastView: 'daily'
    }
  },
  methods: {
    fetchWeather (e) {
      if (e.key == "Enter") {
        this.fetchCurrentWeather();
        this.fetchDailyForecast();
        // this.fetchHourlyForecast();
      }
    },
    fetchCurrentWeather () {
      fetch(`${this.url_base}weather?q=${this.query}&units=imperial&APPID=${this.api_key}`)
        .then(response => {
          return response.json();
        }).then(this.setCurrentWeather);
    },
    fetchDailyForecast () {
      fetch(`${this.url_base}forecast/daily?q=${this.query}&cnt=6&units=imperial&appid=${this.api_key}`)
        .then(response => {
          return response.json();
        }).then(this.setDailyForecast);
    },
    fetchHourlyForecast () {
      fetch(`${this.url_base}forecast/hourly?q=${this.query}&units=imperial&appid=${this.api_key}`)
        .then(response => {
          return response.json();
        }).then(this.setDailyForecast);
    },
    setCurrentWeather (results) {
      this.weather = results;
      console.log('test');
      fetch(`${this.url_base}onecall?lat=${this.weather.coord.lat}&lon=${this.weather.coord.lon}&units=imperial&appid=${this.api_key}`)
        .then(response => {
          return response.json();
        }).then(this.setHourlyForecast);
    },
    setDailyForecast (results) {
      this.dailyForecast = results;
    },
    setHourlyForecast (results) {
      this.hourlyForecast = results.hourly
    },
    moment,
    dateBuilder () {
      let d = new Date();
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
      let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();

      return `${day} ${date} ${month} ${year}`;
    },
    viewHourlyForecast () {
      this.forecastView = 'hourly';
    },
    viewDailyForecast () {
      this.forecastView = 'daily';
    }
  }
}
</script>
