<template>
  <div ref="map" class="map"></div>
</template>

<script lang="ts">
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'
import axios from 'axios'

const limit = 30
const apiUrlStops = `https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records?limit=${limit}`
const apiUrlLines = `https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-circuits/records?where=route_type%20LIKE%20%22Tram%22%20OR%20%22Bus%22&limit=${limit}`

export default {
  data() {
    return {
      busStops: [],
      tanLines: [],
      busLines: [],
      tramLines: [],
      map: null
    }
  },
  async created() {
    try {
      const stops = await axios.get(apiUrlStops)
      this.busStops = stops.data.results
      const lines = await axios.get(apiUrlLines)
      this.tanLines = lines.data.results

      this.tanLines.forEach((tanLine) => {
        if (tanLine.route_type == 'Tram') {
          this.tramLines.push(tanLine)
        }
        if (tanLine.route_type == 'Bus') {
          this.busLines.push(tanLine)
        }
      })
      this.initMap()
    } catch (error) {
      console.log(error)
    }
  },
  methods: {
    initMap() {
      this.map = L.map(this.$refs.map).setView([47.2184, -1.5536], 12)

      // Draw lines
      this.tramLines.forEach((tramLine) => {
        const coordinates = tramLine.shape.geometry.coordinates
        //console.log(coordinates)
        //console.log(tramLine.route_color, latLngs)
        coordinates.forEach((coords) => {
          coords.forEach((coord) => {
            coord.reverse()
          })
          L.polyline(coords, { color: `#${tramLine.route_color}`, stroke: true }).addTo(this.map)
        })
      })

      // Draw lines
      this.busLines.forEach((busLine) => {
        const coordinates = busLine.shape.geometry.coordinates
        //console.log(coordinates)
        //console.log(busLine.route_color, latLngs)
        coordinates.forEach((coords) => {
          coords.forEach((coord) => {
            coord.reverse()
          })
          L.polyline(coords, { color: `#${busLine.route_color}`, stroke: true }).addTo(this.map)
        })
      })

      this.busStops.forEach((stop, index) => {
        const NOM_ARRET = `Nom de l'arrêt&nbsp;: ${stop.stop_name}`
        const ACCES_HANDICAPE = stop.wheelchair_boarding
          ? 'Accès handicapé&nbsp;:' + stop.wheelchair_boarding
          : 'Aucun accès handicapé'
        const TYPE_LIGNE =
          stop.location_type == 0 ? 'Type de ligne&nbsp;: Bus' : 'Type de ligne&nbsp;: Tram'

        const iconUrl = stop.location_type === 0 ? 'src/assets/bus.png' : 'src/assets/tram.png'
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
            `${TYPE_LIGNE}<br>${NOM_ARRET}<br>${ACCES_HANDICAPE}<br><img src="${stop.imageUrl}" alt="Image Mapillary" width="200" height="100"/>`
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
      }).addTo(this.map)
    }
  }
}
</script>

<style scoped>
.map {
  height: 70vh;
  max-height: 700px;
  width: 95vw;
  max-width: 1200px;
}
</style>
