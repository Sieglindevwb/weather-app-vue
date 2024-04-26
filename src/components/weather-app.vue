<template>
  <div class="weather-container">
    <div class="weather-wrap">
      <div class="search-box">
        <input
          type="text"
          placeholder="Search..."
          class="search-bar"
          v-model="query"
          v-on:keypress="fetchWeather"
        />
      </div>
      <div class="weather-info" v-if="typeof weather.main !== 'undefined'">
        <div class="location-box">
          <div class="location">
            {{ weather.name }},{{ weather.sys.country }}
          </div>
          <div class="date">{{ todaysDate() }}</div>
        </div>
        <div class="weather-box">
          <div class="temp">{{ Math.round(weather.main.temp) }}°c</div>
          <div class="weather">{{ weather.weather[0].main }}</div>
          <div class="icon">
            <img :src="`${weather_icon}${weather.weather[0].icon}@2x.png`" />
          </div>
        </div>
        <div class="daily-weather" v-if="typeof forecast !== 'undefined'">
          <div
            v-for="dayForecast in groupedForecast"
            :key="dayForecast.date"
            class="forecast-item"
          >
            <p>{{ dayForecast.date }}</p>
            <p>{{ Math.round(dayForecast.avgTemp) }}°C</p>
            <p>{{ dayForecast.weather }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  data() {
    return {
      api_key: process.env.VUE_APP_API_KEY,
      url_base: 'https://api.openweathermap.org/data/2.5/',
      weather_icon: 'http://openweathermap.org/img/wn/',
      query: '',
      weather: {},
      forecast: {
        list: [], // Initialize 'list' property as an empty array
      },
      latitude: null,
      longitude: null,
    };
  },
  methods: {
    async fetchWeather(e) {
      if (e.key == 'Enter') {
        let responseWeather, responseForecast;
        if (this.query) {
          // Fetch current weather by city name
          responseWeather = await axios.get(
            `${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`
          );
          // Fetch weather forecast for the next 5 days by city name
          responseForecast = await axios.get(
            `${this.url_base}forecast?q=${this.query}&units=metric&APPID=${this.api_key}`
          );
        } else if (this.latitude && this.longitude) {
          // Fetch current weather and weather forecast by latitude and longitude
          responseWeather = await axios.get(
            `${this.url_base}weather?lat=${this.latitude}&lon=${this.longitude}&units=metric&APPID=${this.api_key}`
          );
          responseForecast = await axios.get(
            `${this.url_base}forecast?lat=${this.latitude}&lon=${this.longitude}&units=metric&APPID=${this.api_key}`
          );
        } else {
          // Handle case where neither latitude and longitude nor city name is provided
          console.error(
            'Please provide either city name or latitude and longitude.'
          );
          return;
        }
        console.log('Forecast data:', responseForecast.data);
        // Set results for current weather
        this.setResults(responseWeather.data);
        // Set forecast data
        this.setForecast(responseForecast.data);
        console.log('Forecast object:', this.forecast);
      }
    },
    setResults(returnedResponse) {
      this.weather = returnedResponse;
    },
    setForecast(forecastData) {
      this.forecast = forecastData;
    },
    todaysDate() {
      const months = [
        'Jan',
        'Feb',
        'Mar',
        'Apr',
        'May',
        'Jun',
        'Jul',
        'Aug',
        'Sep',
        'Oct',
        'Nov',
        'Dec',
      ];
      const days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
      let d = new Date();
      let month = months[d.getMonth()];
      let day = days[d.getDay()];
      let date = d.getDate();
      let year = d.getFullYear();
      return `${month} ${date} ${day} ${year}`;
    },
    formatDate(timestamp) {
      const date = new Date(timestamp * 1000); // Convert timestamp to milliseconds
      const day = date.getDate();
      const month = date.getMonth() + 1;
      const year = date.getFullYear();
      return `${day}/${month}/${year}`;
    },
  },
  computed: {
    groupedForecast() {
      // Group forecast data by day
      const grouped = {};
      this.forecast.list.forEach((item) => {
        const date = this.formatDate(item.dt);
        if (!grouped[date]) {
          grouped[date] = {
            date: date,
            temps: [],
            weather: item.weather[0].main,
          };
        }
        grouped[date].temps.push(item.main.temp);
      });

      // Log the grouped data after it's initialized
      console.log('Grouped forecast:', grouped);

      // Calculate average temperature for each day
      return Object.values(grouped).map((dayForecast) => {
        const totalTemp = dayForecast.temps.reduce(
          (acc, temp) => acc + temp,
          0
        );
        return {
          date: dayForecast.date,
          avgTemp: totalTemp / dayForecast.temps.length,
          weather: dayForecast.weather,
        };
      });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@300;500;700;900&display=swap');
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Montserrat';
}
.weather-container {
  background-image: url('../assets/wolkjes.jpg');
  background-size: cover;
  background-position: center;
  transition: 0.4s;
  width: 375px;
  margin: 0 auto;
  border-radius: 25px;
  margin-top: 50px;
  box-shadow: 0px 0px 30px #00000065;
}
.weather-wrap {
  height: 600px;
  padding: 25px;
  border-radius: 25px;
  background-image: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0.15),
    rgba(0, 0, 0, 0.4)
  );
}
.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  color: #313131;
  font-size: 20px;
  appearance: none;
  border: none;
  outline: none;
  background: none;
  background-color: rgba(255, 255, 255, 0.5);
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  border-radius: 10px;
  transition: 0.4s;
}
.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
}
.location-box .location {
  color: #fff;
  font-size: 32px;
  font-weight: 500;
  font-style: italic;
  text-align: center;
  margin-top: 30px;
}
.location-box .date {
  color: #fff;
  font-size: 20px;
  font-weight: 300;
  text-align: center;
}
.weather-box {
  text-align: center;
}
.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: #fff;
  font-size: 102px;
  font-weight: 900;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 30px 0px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  font-style: italic;
}
.weather-box .weather {
  color: #fff;
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.daily-weather {
  color: black;
  display: flex;
  align-items: center;
}
.forecast-item {
  background-color: rgba(255, 255, 255, 0.2);
  padding: 10px;
  margin-bottom: 10px;
  border-radius: 8px;
}

.forecast-date,
.forecast-temp,
.forecast-weather {
  color: #fff; /* Text color same as other elements */
}
</style>
