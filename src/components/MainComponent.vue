<template>
  <div>
    <p>CURRENT WEATHER</p>
    <p>
      {{ weatherData.temperature }}°
      {{ temperatureType === "Celsius" ? "C" : "F" }} |
      {{ weatherData.weatherMode }}
    </p>
    <p>{{ address.toLocaleUpperCase() }}</p>
    <p>HUMIDITY {{ weatherData.humidity }}%</p>
    <p>WIND {{ weatherData.wind }} M/S</p>
    <p>
      SUNRISE {{ new Date(weatherData.sunrise * 1000).getHours() }}:{{
        new Date(weatherData.sunrise * 1000).getMinutes()
      }}
    </p>
    <p>
      SUNSET {{ new Date(weatherData.sunset * 1000).getHours() }}:{{
        new Date(weatherData.sunset * 1000).getMinutes()
      }}
    </p>
  </div>
</template>

<script setup>
import { onMounted, ref, reactive, computed } from "vue";

const weatherData = reactive({
  weatherMode: "CLOUDS",
  coordinate: {
    lat: 51.51,
    lon: -0.12,
  },
  temperature: 0,
  humidity: 0,
  wind: 0,
  sunrise: 0,
  sunset: 1666459919802,
  country: "UK",
  city: "London",
});
const temperatureType = ref("Fahrenheit"); // Fahrenheit/Celsius
const address = computed(() => `${weatherData.city}, ${weatherData.country}`);

onMounted(() => {
  getCoordinate();
  // setTimeout(fetchData(), 1000);
  // setTimeout(getCountryName, 100);
});

const fetchData = () => {
  if (localStorage.getItem("weatherData")) {
    const data = JSON.parse(localStorage.getItem("weatherData"));
    if (
      data.coordinate.lat.toFixed(2) !== weatherData.coordinate.lat ||
      data.coordinate.lon.toFixed(2) !== weatherData.coordinate.lon ||
      new Date().getTime() - new Date(data.date).getTime() > 180
    ) {
      console.log("call network to get weather data");
      getCountryName();
      getWeatherData();
    }
  } else {
    getCountryName();
    getWeatherData();
  }
};

const getCountryName = () => {
  fetch(`http://www.geoplugin.net/json.gp`)
    .then((res) => res.json())
    .then((data) => {
      weatherData.city = data.geoplugin_city;
      weatherData.country = data.geoplugin_countryName;
    })
    .catch(() => alert("Error in getCountryName"));
};

const getCoordinate = () => {
  const success = (position) => {
    weatherData.coordinate.lat = position.coords.latitude.toFixed(2);
    weatherData.coordinate.lon = position.coords.longitude.toFixed(2);
    fetchData();
  };
  const error = (err) => {
    alert(`ERROR(${err.code}): ${err.message}`);
  };
  const options = {
    enableHighAccuracy: true,
    timeout: 5000,
    maximumAge: 0,
  };
  navigator.geolocation.getCurrentPosition(success, error, options);
};

const getWeatherData = async () => {
  // if()
  fetch(
    `https://api.openweathermap.org/data/2.5/weather?lat=${weatherData.coordinate.lat}&lon=${weatherData.coordinate.lon}&units=Imperial&appid=5ae1a521a60a1bbba2a698d1a3466cb6`
  )
    .then((response) => response.json())
    .then((data) => {
      weatherData.weatherMode = data.weather[0].main;
      weatherData.coordinate = {
        lat: Number(data.coord.lat.toFixed(2)),
        lon: Number(data.coord.lon.toFixed(2)),
      };
      weatherData.temperature = data.main.temp;
      weatherData.humidity = data.main.humidity;
      weatherData.wind = data.wind.speed;
      weatherData.sunrise = data.sys.sunrise;
      weatherData.sunset = data.sys.sunset;
      // console.log("data", data);
      console.log("weatherData", weatherData);
      localStorage.setItem(
        "weatherData",
        JSON.stringify({
          date: new Date(),
          ...weatherData,
        })
      );
    })
    .catch((err) => alert(`ERROR IN DATA FETCHING:${err.message}`));
};
</script>
