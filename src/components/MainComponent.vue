<template>
  <div class="main-container">
    <div class="d-flex">
      <p>{{ address.toLocaleUpperCase() }}</p>
    </div>
    <div class="d-flex">
      <h3>
        {{ calculatedTemperature(temperatureType, weatherData.temperature) }}°
        {{ temperatureType === "Celsius" ? "C" : "F" }} |
        {{ weatherData.weatherMode }}
      </h3>
    </div>
    <div class="d-flex">
      <img :src="dynamicWeatherIcon" alt="img" />
    </div>
    <div class="mt-20 flex-between">
      <p>HUMIDITY</p>
      <p>{{ weatherData.humidity }}%</p>
    </div>
    <div class="flex-between">
      <p>WIND</p>
      <p>{{ weatherData.wind }} M/S</p>
    </div>
    <div class="flex-between">
      <p>SUNRISE</p>
      <p>{{ sunrise }}</p>
    </div>
    <div class="flex-between">
      <p>SUNSET</p>
      <p>{{ sunset }}</p>
    </div>
    <div class="mt-20 d-flex">
      <button
        class="btn"
        @click="
          temperatureType =
            temperatureType === 'Fahrenheit' ? 'Celsius' : 'Fahrenheit'
        "
      >
        Convert to
        {{ temperatureType === "Fahrenheit" ? "Celsius" : "Fahrenheit" }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { onBeforeMount, ref, reactive, computed } from "vue";
import { countryCodeToCountryName, calculatedTemperature } from "@/utils";
// define app state variables
const weatherData = reactive({
  weatherMode: "Loading...",
  weatherIcon: "Loading...",
  coordinate: {
    lat: 40.73,
    lon: -73.93,
  },
  temperature: "Loading...",
  humidity: "Loading...",
  wind: "Loading...",
  sunrise: 0,
  sunset: 0,
  country: "Loading...",
  city: "Loading...",
});
const temperatureType = ref("Fahrenheit"); // Fahrenheit/Celsius
const address = computed(() => `${weatherData.city}, ${weatherData.country}`);

// it's return sunrise local time from given unix UTC time
const sunrise = computed(
  () =>
    `${new Date(weatherData.sunrise * 1000).getHours()}:${new Date(
      weatherData.sunrise * 1000
    ).getMinutes()}`
);

// same as above
const sunset = computed(
  () =>
    `${new Date(weatherData.sunset * 1000).getHours()}:${new Date(
      weatherData.sunset * 1000
    ).getMinutes()}`
);

// weather mode = rain/snow/sunny/cloudy/windy
// our api give us icon code for specific weather
// here i just use the icon code for display related icon from static folder
// it's return the corresponding static image name
const dynamicWeatherIcon = computed(() => {
  let icon = weatherData.weatherIcon.substring(0, 2);
  let str = "";
  switch (icon) {
    case "01":
      str = "sunny";
      break;
    case "02":
    case "03":
    case "04":
      str = "cloudy";
      break;
    case "09":
    case "10":
    case "11":
      str = "rainy";
      break;
    case "13":
      str = "snow";
      break;
    case "50":
      str = "windy";
      break;
    default:
      break;
  }
  return `${str}.svg`;
});

onBeforeMount(() => getCoordinate());

const getCoordinate = () => {
  const success = (position) => {
    weatherData.coordinate.lat = position.coords.latitude.toFixed(2);
    weatherData.coordinate.lon = position.coords.longitude.toFixed(2);
    fetchData();
  };
  const error = (err) => {
    alert(`ERROR(${err.code}): ${err.message}`);
    getLatLonManually();
  };
  const options = {
    enableHighAccuracy: true,
    timeout: 5000,
    maximumAge: 0,
  };
  navigator.geolocation.getCurrentPosition(success, error, options);
};

// if user is denied to access the location
const getLatLonManually = () => {
  try {
    fetch(`http://www.geoplugin.net/json.gp`)
      .then((res) => res.json())
      .then((data) => {
        weatherData.coordinate.lat = parseFloat(
          data.geoplugin_latitude
        ).toFixed(2);
        weatherData.coordinate.lon = parseFloat(
          data.geoplugin_longitude
        ).toFixed(2);
        fetchData();
      });
  } catch (error) {
    alert(`Error in geoPlugin: ${error.message}`);
  }
};

// before fetching just checking localStorage data is valid to use
const fetchData = () => {
  if (localStorage.getItem("weatherData")) {
    const data = JSON.parse(localStorage.getItem("weatherData"));
    if (
      data.coordinate.lat.toFixed(2) !== weatherData.coordinate.lat ||
      data.coordinate.lon.toFixed(2) !== weatherData.coordinate.lon ||
      new Date().getTime() - new Date(data.date).getTime() > 1800000
    ) {
      // cross checking with localStorage
      // if localStorage latitude & present latitude is not same
      // or longitude longitude & present longitude is not same
      // or last api call time & present time difference is greater than 30 minutes
      // any one of the above conditions is true thn call the data fetch api
      getWeatherData();
    } else {
      // if localStorage data available & valid to use
      // retrieve data from localStorage to reactive object
      weatherData.weatherMode = data.weatherMode;
      weatherData.weatherIcon = data.weatherIcon;
      weatherData.temperature = data.temperature;
      weatherData.humidity = data.humidity;
      weatherData.wind = data.wind;
      weatherData.sunrise = data.sunrise;
      weatherData.sunset = data.sunset;
      weatherData.country = data.country;
      weatherData.city = data.city;
    }
  } else {
    getWeatherData();
  }
};

// Right now  https://darksky.net/dev/ not accepting new signup
// Get weather data from https://api.openweathermap.org
const getWeatherData = async () => {
  try {
    fetch(
      `https://api.openweathermap.org/data/2.5/weather?lat=${weatherData.coordinate.lat}&lon=${weatherData.coordinate.lon}&units=Imperial&appid=5ae1a521a60a1bbba2a698d1a3466cb6`
    )
      .then((response) => response.json())
      .then((data) => {
        // store response data into reactive state object
        weatherData.weatherMode = data.weather[0].main;
        weatherData.weatherIcon = data.weather[0].icon;
        weatherData.coordinate = {
          lat: Number(data.coord.lat.toFixed(2)),
          lon: Number(data.coord.lon.toFixed(2)),
        };
        weatherData.temperature = data.main.temp;
        weatherData.humidity = data.main.humidity;
        weatherData.wind = data.wind.speed;
        weatherData.sunrise = data.sys.sunrise;
        weatherData.sunset = data.sys.sunset;
        weatherData.city = data.name;
        weatherData.country = data.sys.country;
        weatherData.country = countryCodeToCountryName(data.sys.country);
        // save data into localStorage for future use
        localStorage.setItem(
          "weatherData",
          JSON.stringify({
            date: new Date(),
            ...weatherData,
          })
        );
      });
  } catch (err) {
    alert(`ERROR IN DATA FETCHING:${err.message}`);
  }
};
</script>

<style lang="scss" scoped>
@import "@/assets/style.scss";
.main-container {
  background-color: #26282c;
  color: #fff;
  padding: 20px;
  border-radius: 20px;
  min-width: 200px;
}
</style>
