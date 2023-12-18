<template>
    <div ref="map" class="map"></div>
</template>
  
<script>// lang="ts"
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import axios from 'axios';

const apiUrlStops = "https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records";
const apiUrlLines = "https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-circuits/records?where=route_type%20LIKE%20%22Tram%22%20OR%20%22Bus%22&limit=100";
const token = 'MLY|6817262911684044|dd678e977d02b8ba219b398d2cd70fba';
const imagesIds = ['1354489271597701', 'src/assets/adrienne_bolland.jpg', '2561170414179121', '5620189844688123', 'src/assets/les_anges.jpg', 'src/assets/ampere.jpg', 'src/assets/aquitaine.jpg', 'src/assets/arbois.jpg', 'src/assets/auge.jpg', 'src/assets/barbiniere.jpg'];

export default {
data() {
    return {
    //arrets: [],
    busStops: [],
    tanLines: [],
    busLines: [],
    tramLines: [],
    map: null,
    //nombreArrets: 0,
    };
},
async created() {
    try {
    //await this.getStops();
    //this.filterStopsByName();
    const stops = await axios.get(apiUrlStops);
    this.busStops = stops.data.results;
    const lines = await axios.get(apiUrlLines);
    this.tanLines = lines.data.results;

    this.tanLines.forEach(tanLine => {
        if (tanLine.route_type == 'Tram') {
        this.tramLines.push(tanLine);
        }
        if (tanLine.route_type == 'Bus') {
        this.busLines.push(tanLine);
        }
    });
    let index = 0;
    for (const stop of this.busStops) {
      stop.imageId = imagesIds[index];
      if (stop.imageId.substr(0, 3) != 'src') {
        console.log("enter " + stop.imageId);
        const apiUrlImage = `https://graph.mapillary.com/${stop.imageId}?access_token=${token}&fields=thumb_256_url&client_id=6817262911684044`;
        const response = await axios.get(apiUrlImage);
        stop.imageUrl = response.data.thumb_256_url;
      } else {
        stop.imageUrl = stop.imageId;
      }
      index = index + 1;
    }

    this.initMap();
    } catch (error) {
    console.log(error);
    }
},
methods: {
  /*async getStops() {
      await axios
        .get("https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records")
        .then((result) => {
          this.nombreArrets = result.data.total_count;
        });

      for (let i = 0; i <= this.nombreArrets; i += 100) {
        await axios
          .get(`https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records?offset=${i}&limit=100`)
          .then((response) => {
            let tab: any[] = response.data.results;
            tab.forEach(e => this.arrets.push(e));
          });
      }
    },

    async filterStopsByName() {
  const stopsMap = new Map();

  this.arrets.forEach((arret) => {
    const key = arret.stop_name;

    if (!stopsMap.has(key)) {
      stopsMap.set(key, [arret]);
    } else {
      const existingArrets = stopsMap.get(key);
      const hasDifferentLocationType = existingArrets.every(existingArret => existingArret.location_type !== arret.location_type);

      if (hasDifferentLocationType) {
        existingArrets.push(arret);
      }
    }
  });
  this.arrets = Array.from(stopsMap.values()).flat();
},*/

    initMap() {
      this.map = L.map(this.$refs.map).setView([47.2184, -1.5536], 12);

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
        const TYPE_LIGNE = stop.location_type == 0 ? 'Type de ligne&nbsp;: Bus' : 'Type de ligne&nbsp;: Tram';
        const iconUrl = stop.location_type === 0 ? 'src/assets/bus.png' : 'src/assets/tram.png';
        const marker = L.marker([stop.stop_coordinates.lat, stop.stop_coordinates.lon], {
            icon: L.icon({
            iconUrl,
            iconSize: [25, 25],
            iconAnchor: [12, 41],
            popupAnchor: [1, -34],
            shadowSize: [41, 41]
            })
  })
  .bindPopup(`${TYPE_LIGNE}<br>${NOM_ARRET}<br>${ACCES_HANDICAPE}<br><img src="${stop.imageUrl}" alt="Image Mapillary" width="200" height="100"/>`)
  marker.on('click', () => {
      this.map.flyTo([stop.stop_coordinates.lat, stop.stop_coordinates.lon], 16, {
          duration: 1,
          easeLinearity: 0.2,
      });
  })
  .addTo(this.map);
});



  
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
          maxZoom: 19,
          minZoom: 8
        }).addTo(this.map)
    },
},
};
</script>

<style scoped>
.map {
height: 400px;
}
</style>  