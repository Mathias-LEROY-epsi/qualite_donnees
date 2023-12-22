<template>
  <div class="searchArea">
    <div class="container">
      <input
        class="searchBar"
        type="search"
        v-model="searchString"
        @change="search"
        :placeholder="`Recherchez des ${searchType === 'stops' ? 'arrêts' : 'lignes'}`" />
      <button @onClick="search" class="searchBtn">Rechercher</button>
    </div>
    <div class="container">
      <button @click="toggleSearchType" class="toggle">
        Autre recherche : {{ searchType === 'stops' ? 'lignes' : 'arrêts' }}
      </button>
      <div v-if="searchType === 'stops'">
        <select v-model="selectedElement" @change="selectElement(selectedElement)">
          <option v-for="(stop, index) in busStops" :key="index" :value="stop">
            {{ stop.stop_name }}
          </option>
        </select>
      </div>
      <div v-else>
        <select v-model="selectedElement" @change="selectElement(selectedElement)">
          <option v-for="(line, index) in busLines" :key="index" :value="line">
            {{ line.route_long_name }}
          </option>
        </select>
      </div>
    </div>
    <p>{{ this.selectedElement }}</p>
    <div class="legend">
      <LegendComponent />
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import LegendComponent from '@/components/LegendComponent.vue'

export default {
  name: 'SearchBar',
  components: {
    LegendComponent
  },
  data() {
    return {
      searchString: '',
      selectedElement: null,
      searchType: 'stops',
      busStops: [],
      busLines: [],
      map: null
    }
  },
  methods: {
    toggleSearchType() {
      this.searchType = this.searchType === 'stops' ? 'lines' : 'stops'
    },
    search() {
      if (this.searchType === 'stops') {
        this.searchStops()
      } else {
        this.searchLines()
      }
    },
    selectElement(element) {
      this.$emit('element-selected', element)
    },
    async searchStops() {
      const search = this.searchString.trim().toLowerCase()
      return await axios
        .get(
          `https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records?where=%22${search}%22&limit=-1`
        )
        .then((res) => {
          this.busStops = res.data.results
          console.log(this.busStops)
        })
    },
    async searchLines() {
      const search = this.searchString.trim().toLowerCase()
      return await axios
        .get(
          `https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-circuits/records?where=%22${search}%22&limit=-1`
        )
        .then((res) => {
          this.busLines = res.data.results
          console.log(this.busLines)
        })
    }
  }
}
</script>

<style scoped>
.legend {
  display: flex;
  align-items: flex-start;
  position: absolute;
  bottom: 0;
  right: 0;
  margin: 1em;
}
.toggle {
  background-color: #f5f5f5;
  border: 1px solid #ccc;
  border-radius: 5px;
  width: 200px;
  padding: 0.25em;
  margin: 1em;
}
.searchArea {
  display: flex;
  flex-direction: column;
  justify-content: left;
  align-items: center;
}
.searchBtn {
  background-color: #f5f5f5;
  border: 1px solid #ccc;
  border-radius: 5px;
  width: 100px;
  padding: 0.25em;
}
.searchBar {
  background-color: #f5f5f5;
  border: 1px solid #ccc;
  border-radius: 5px;
  width: 250px;
  padding: 0.25em;
}
.container {
  display: flex;
  gap: 1em;
  justify-content: center;
  align-items: center;
  width: 500px;
}
</style>
