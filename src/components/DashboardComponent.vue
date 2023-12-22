<template>
  <div class="dashboard">
    <div class="mapArea">
      <div ref="map" class="map"></div>
      <div class="legend">
        <div class="row">
          <div class="col-4 inline">
            <span class="legend-icon"> <img src="src/assets/bus.png" alt="icon de bus" /></span> Bus
          </div>
          <div class="col-4 inline">
            <span class="legend-icon"> <img src="src/assets/tram.png" alt="icon de tram" /></span>
            Tramway
          </div>
          <div class="col-4 inline"><span class="legend-icon path-icon"></span> Trajet</div>
        </div>
      </div>
    </div>

    <div class="searchArea">
      <div class="container">
        <input
          class="searchBar"
          type="search"
          v-model="searchString"
          @change="search"
          :placeholder="`Recherchez des ${isLineSearch ? 'lignes' : 'arrêts'}`" />
        <button @onClick="search" class="searchBtn">Rechercher</button>
      </div>
      <div class="container">
        <div class="toggleArea">
          <label>Arrêts</label>
          <div class="form-check form-switch">
            <input
              class="form-check-input"
              type="checkbox"
              role="switch"
              id="flexSwitchCheckChecked"
              v-model="isLineSearch"
              @change="resetSearchBar" />
            <label class="form-check-label" for="flexSwitchCheckChecked">Lignes</label>
          </div>
          <div v-if="isLineSearch">
            <select v-model="selectedElement" @change="selectLine(selectedElement)">
              <option v-for="(line, index) in searchTramResult" :key="index" :value="line">
                {{ line.route_long_name }}
              </option>
            </select>
          </div>
          <div v-else>
            <select v-model="selectedElement" @change="selectStop(selectedElement)">
              <option v-for="(stop, index) in searchBusResult" :key="index" :value="stop">
                {{ stop.stop_name }}
              </option>
            </select>
          </div>
        </div>
      </div>
      <!-- <ul>
        <li>Zoom: {{ mapProps.zoom }}</li>
        <li>Center: {{ mapProps.center }}</li>
        <li>Bounds: {{ mapProps.bounds }}</li>
        <li>searchString : {{ searchString }}</li>
        <li>
          selectedElement:
          {{
            isLineSearch ? selectedElement?.stop_name : selectedElement?.route_long_name
          }}
        </li>
      </ul> -->
    </div>
  </div>
</template>

<script lang="ts">
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'
import axios from 'axios'

const apiUrlLines =
  'https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-circuits/records?where=route_type%20LIKE%20%22Tram%22%20OR%20%22Bus%22&limit=30'
const token = 'MLY|6817262911684044|dd678e977d02b8ba219b398d2cd70fba'
const imagesIds = [
  { name: 'Abel Durand', value: '1354489271597701' },
  { name: 'Adrienne Bolland', value: 'src/assets/adrienne_bolland.jpg' },
  { name: 'Aimé Delrue', value: '2561170414179121' },
  { name: 'Avenue des Pins', value: '5620189844688123' },
  { name: 'Les Anges', value: 'src/assets/les_anges.jpg' },
  { name: 'Ampère', value: 'src/assets/ampere.jpg' },
  { name: 'Aquitaine', value: 'src/assets/aquitaine.jpg' },
  { name: 'Arbois', value: 'src/assets/arbois.jpg' },
  { name: 'Augé', value: 'src/assets/auge.jpg' },
  { name: 'Barbinière', value: 'src/assets/barbiniere.jpg' }
]

export default {
  data() {
    return {
      stops: [],
      tanLines: [],
      busLines: [],
      tramLines: [],
      map: null,
      mapProps: {
        zoom: 12,
        center: null,
        bounds: null
      },
      nombreArrets: 0,

      searchString: '',
      selectedElement: null,
      isLineSearch: false,
      searchBusResult: [],
      searchTramResult: []
    }
  },
  async created() {
    try {
      // Récupération des lignes de bus et tram
      const lines = await axios.get(apiUrlLines)
      this.tanLines = lines.data.results

      // Filtrage des lignes de tram et de bus
      this.tanLines.forEach((tanLine) => {
        if (tanLine.route_type == 'Tram') {
          this.tramLines.push(tanLine)
        }
        if (tanLine.route_type == 'Bus') {
          this.busLines.push(tanLine)
        }
      })
      await this.getStops()
      // Récupération des photos (via l'API Mapillary si possible)
      await this.addStopsPhoto()
      // Initialisation de la carte
      this.initMap()
    } catch (error) {
      console.log(error)
    }
  },
  methods: {
    resetSearchBar() {
      this.searchBusResult = []
      this.searchTramResult = []
      this.selectedElement = null
    },
    toggleSearchType() {
      this.resetSearchBar()
      this.isLineSearch = !this.isLineSearch
    },
    async search() {
      if (this.searchString === '') {
        this.resetSearchBar()
      }
      if (this.isLineSearch) {
        const search = this.searchString.trim().toLowerCase()
        return await axios
          .get(
            `https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-circuits/records?where=%22${search}%22&limit=-1`
          )
          .then((res) => {
            const newResults = res.data.results

            this.searchTramResult.push(...newResults)
          })
          .then(() => {
            console.log(this.searchTramResult)
            return this.searchTramResult
          })
      } else {
        const search = this.searchString.trim().toLowerCase()
        const seenStopIds = new Set()
        return await axios
          .get(
            `https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records?where=%22${search}%22&limit=-1`
          )
          .then((res) => {
            const newResults = res.data.results.filter((result) => {
              if (seenStopIds.has(result.stop_name)) {
                return false
              } else {
                seenStopIds.add(result.stop_name)
                return true
              }
            })

            this.searchBusResult.push(...newResults)
          })
          .then(() => {
            console.log(this.searchBusResult)
            return this.searchBusResult
          })
      }
    },
    selectStop(element) {
      this.$emit('element-selected', element)

      this.map.flyTo(
        [this.selectedElement.stop_coordinates.lat, this.selectedElement.stop_coordinates.lon],
        16,
        {
          duration: 1,
          easeLinearity: 0.2
        }
      )
    },
    selectLine(element) {
      this.$emit('element-selected', element)

      /* this.map.flyTo(
        [this.selectedElement.stop_coordinates.lat, this.selectedElement.stop_coordinates.lon],
        16,
        {
          duration: 1,
          easeLinearity: 0.2
        }
      ) */
    },
    // Récupération des arrêts
    async getStops() {
      const seenStopIds = new Set()
      await axios
        .get(
          'https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records'
        )
        .then(() => {
          //this.nombreArrets = result.data.total_count;
          this.nombreArrets = 300
        })

      try {
        for (let i = 0; i <= this.nombreArrets; i += 100) {
          await axios
            .get(
              `https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records?offset=${i}&limit=100`
            )
            /* .then((res) => {
              let tab: any[] = res.data.results
              tab.forEach((e) => {
                this.stops.push(e)
              })
            }) */
            .then((res) => {
              const newResults = res.data.results.filter((result) => {
                if (seenStopIds.has(result.stop_id)) {
                  return false
                } else {
                  seenStopIds.add(result.stop_id)
                  return true
                }
              })

              this.stops.push(...newResults)
            })
            .then(() => {
              console.log(this.stops)
              return this.searchBusResult
            })
        }
      } catch (error) {
        console.log(error)
      }
    },

    // Initialisation de la carte
    initMap() {
      this.map = L.map(this.$refs.map).setView([47.2184, -1.5536], 12)
      const map = this.map

      map.on('moveend', () => {
        this.mapProps.center = map.getCenter()
        this.mapProps.bounds = map.getBounds()
      })

      map.on('zoomend', async () => {
        this.mapProps.zoom = map.getZoom()
      })

      // Dessin des lignes de tram
      this.tramLines.forEach((tramLine) => {
        const coordinates = tramLine.shape.geometry.coordinates
        coordinates.forEach((coords) => {
          coords.forEach((coord) => {
            coord.reverse()
          })
          L.polyline(coords, { color: `#${tramLine.route_color}`, stroke: true }).addTo(this.map)
        })
      })

      // Dessin des lignes de bus
      this.busLines.forEach((busLine) => {
        const coordinates = busLine.shape.geometry.coordinates
        coordinates.forEach((coords) => {
          coords.forEach((coord) => {
            coord.reverse()
          })
          L.polyline(coords, { color: `#${busLine.route_color}`, stroke: true }).addTo(this.map)
        })
      })

      // Ajout des marqueurs pour les arrêts de bus
      this.stops.forEach((stop) => {
        const NOM_ARRET = `Nom de l'arrêt&nbsp;: ${stop.stop_name}`
        const ACCES_HANDICAPE = stop.wheelchair_boarding
          ? 'Accès handicapé&nbsp;:' + stop.wheelchair_boarding
          : 'Aucun accès handicapé'
        const TYPE_LIGNE =
          stop.location_type == 1 ? 'Type de ligne&nbsp;: Bus' : 'Type de ligne&nbsp;: Tram'
        const iconUrl = stop.location_type == 1 ? 'src/assets/bus.png' : 'src/assets/tram.png'
        L.marker([stop.stop_coordinates.lat, stop.stop_coordinates.lon], {
          icon: L.icon({
            iconUrl,
            iconSize: [25, 25],
            iconAnchor: [12, 41],
            popupAnchor: [1, -34],
            shadowSize: [41, 41]
          })
        })
          .bindPopup(
            `${TYPE_LIGNE}<br>${NOM_ARRET}<br>${ACCES_HANDICAPE}<br><img src="${stop.imageUrl}" alt="Image Mapillary indisponible" width="200" height="100"/>`
          )
          .on('click', () => {
            this.map.flyTo([stop.stop_coordinates.lat, stop.stop_coordinates.lon], 16, {
              duration: 1,
              easeLinearity: 0.2
            })
          })
          .addTo(this.map)
      })

      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 19,
        minZoom: 8
      }).addTo(map)
    },
    // Ajouter les images sur certains arrets de bus
    async addStopsPhoto() {
      for (const stop of this.stops) {
        try {
          const matchingImage = imagesIds.find((image) => image.name === stop.stop_name)
          if (matchingImage) {
            stop.imageId = matchingImage.value

            if (stop.imageId && !stop.imageId.startsWith('src')) {
              const apiUrlImage = `https://graph.mapillary.com/${stop.imageId}?access_token=${token}&fields=thumb_256_url&client_id=6817262911684044`
              const response = await axios.get(apiUrlImage)
              stop.imageUrl = response.data.thumb_256_url
            } else {
              stop.imageUrl = stop.imageId
            }
          } else {
            stop.imageUrl = null
          }
        } catch (error) {
          console.log('Error processing stop:', error)
          stop.imageUrl = null
        }
      }
    }
  }
}
</script>

<style scoped>
* {
  box-sizing: border-box;
}
.dashboard {
  display: flex;
  justify-content: space-between;
  gap: 1em;
}

.mapArea {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.map {
  height: 60vh;
  max-height: 650px;
  width: 60vw;
  max-width: 700px;
}

.toggleArea {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1em;
  margin-top: 1em;
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
  padding-left: 10px;
}
.container {
  display: flex;
  gap: 1em;
  justify-content: center;
  align-items: center;
  width: 500px;
}

.inline {
  display: flex;
  justify-content: center;
  align-items: center;
  padding-block: 0.5em;
}
.legend {
  background-color: white;
  border: 1px solid #ccc;
  padding: 10px;
  margin-top: 10px;
  width: 350px;
}

.legend h2 {
  font-size: 1.5em;
}

.legend-icon {
  display: inline-flex;
  width: 20px;
  margin-right: 5px;
}
.path-icon {
  background-color: black;
  height: 0;
  border: 1px solid black;
}
</style>
