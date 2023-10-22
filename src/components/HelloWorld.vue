<template>
  <div class="greetings">
    <h1 class="Black">{{ msg }}</h1>

    <!-- <div v-if="data">
      <h2>Data from API:</h2>
      <pre>{{ data }}</pre>
    </div> -->
  </div>
</template>

<script setup lang="ts">
import axios from 'axios';

const apiUrl = "https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records?limit=20";
const msg = "Bienvenue,";

const fetchData = async () => {
  try {
    const response = await axios.get(apiUrl);
    //console.log(response)
  } catch (error) {
    console.log(error);
  }
};

fetchData();
</script>

<style scoped>
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  position: relative;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
