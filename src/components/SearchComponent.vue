<template>
  <div class="searchBar">
    <input
      type="search"
      v-model="searchString"
      @change="searchStops()"
      placeholder="Recherchez un arrêt / une ligne" />
    <button @onClick="searchStops()">Rechercher</button>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'SearchBar',
  data() {
    return {
      searchString: '',
      busStops: [],
      busLines: [],
      map: null
    }
  },
  methods: {
    async searchStops() {
      const search = this.searchString.trim().toLowerCase()

      return await axios
        .get(
          `https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records?select=stop_name&where=%22${search}%22&limit=-1`
        )
        .then((res) => {
          this.busStops = res.data.results
          console.log(this.busStops)
        })
    }
    // searchLines: async function () {
    //   const search = this.searchString.trim().toLowerCase()

    //   return await axios.get(`https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-circuits/records?select=route_long_name&where=%22${search}%22&limit=-1`).then((res) => {
    //     this.data = res.data
    //   })
    // },
  }
}
</script>
