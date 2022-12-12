<template>
  <div v-for="city in savedCities" :key="city">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>

  <p v-if="savedCities.length === 0">
    No locations added. To start tracking a location, search in the field above.
  </p>
</template>

<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import CityCard from "./CityCard.vue";

const openWeatherAPIKey = "4896d476dbb705acb9f81fb9c6475de5";
const savedCities = ref([]);
const getCities = async () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));

    const request = [];
    savedCities.value.forEach((city) => {
      request.push(
        axios.get(
          `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lon}&appid=${openWeatherAPIKey}`
        )
      );
    });

    const weatherData = await Promise.all(request);

    // Flicker Delay for AnimationPlaceholder
    await new Promise((res) => setTimeout(res, 1000));

    weatherData.forEach((value, index) => {
      savedCities.value[index].weather = value.data;
    });
  }
};
await getCities();

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: "cityView",
    params: { country: city.country, state: city.state },
    query: { id: city.id, lat: city.coords.lat, lon: city.coords.lon },
  });
};
</script>
