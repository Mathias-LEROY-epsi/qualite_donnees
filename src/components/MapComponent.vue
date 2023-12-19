<template>
  <div ref="map" class="map"></div>
</template>

<script lang="ts">
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import axios from 'axios';

const apiUrlLines =
  "https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-circuits/records?where=route_type%20LIKE%20%22Tram%22%20OR%20%22Bus%22&limit=100";
const token = 'MLY|6817262911684044|dd678e977d02b8ba219b398d2cd70fba';
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
];

export default {
  data() {
    return {
      stops: [],
      tanLines: [],
      busLines: [],
      tramLines: [],
      map: null,
      nombreArrets: 0,
    };
  },
  async created() {
    try {
      await this.getStops();

      // Récupération des lignes de bus et tram
      const lines = await axios.get(apiUrlLines);
      this.tanLines = lines.data.results;

      // Filtrage des lignes de tram et de bus
      this.tanLines.forEach(tanLine => {
        if (tanLine.route_type == 'Tram') {
          this.tramLines.push(tanLine);
        }
        if (tanLine.route_type == 'Bus') {
          this.busLines.push(tanLine);
        }
      });
      // Récupération des photos (via l'API Mapillary si possible)
      await this.addStopsPhoto();
      // Initialisation de la carte
      this.initMap();
    } catch (error) {
      console.log(error);
    }
  },
  methods: {
    // Récupération des arrêts
    async getStops() {
      await axios
        .get(
          "https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records"
        )
        .then((result) => {
          this.nombreArrets = result.data.total_count;
        });

      try {
        for (let i = 0; i <= this.nombreArrets; i += 100) {
          await axios
            .get(
              `https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records?offset=${i}&limit=100`
            )
            .then((response) => {
              let tab: any[] = response.data.results;
              tab.forEach((e) => this.stops.push(e));
            });
        }
      } catch (error) {
        console.log(error);
      }
    },

    // Initialisation de la carte
    initMap() {
      
      this.map = L.map(this.$refs.map).setView([47.2184, -1.5536], 12);
      this.drawLines(this.tramLines);
      this.drawLines(this.busLines);

      // Ajout des marqueurs pour les arrêts de bus et tram
      this.stops.forEach((stop, index) => {
        const NOM_ARRET = `Nom de l'arrêt&nbsp;: ${stop.stop_name}`;
        const ACCES_HANDICAPE = stop.wheelchair_boarding
          ? 'Accès handicapé&nbsp;:' + stop.wheelchair_boarding
          : 'Aucun accès handicapé';
        const TYPE_LIGNE =
          stop.location_type == 0 ? 'Type de ligne&nbsp;: Bus' : 'Type de ligne&nbsp;: Tram';
        const iconUrl = stop.location_type === 0 ? 'src/assets/bus.png' : 'src/assets/tram.png';
        L.marker([stop.stop_coordinates.lat, stop.stop_coordinates.lon], {
          icon: L.icon({
            iconUrl,
            iconSize: [25, 25],
            iconAnchor: [12, 41],
            popupAnchor: [1, -34],
            shadowSize: [41, 41],
          }),
        })
          .bindPopup(
            `${TYPE_LIGNE}<br>${NOM_ARRET}<br>${ACCES_HANDICAPE}<br><img src="${stop.imageUrl}" alt="Image Mapillary indisponible" width="200" height="100"/>`
          )
          .on('click', () => {
            this.map.flyTo([stop.stop_coordinates.lat, stop.stop_coordinates.lon], 16, {
              duration: 1,
              easeLinearity: 0.2,
            });
          })
          .addTo(this.map);
      });
    },
    // Ajouter les images sur certains arrets de bus
    async addStopsPhoto() {
      for (const stop of this.stops) {
        try {
          const matchingImage = imagesIds.find(image => image.name === stop.stop_name);
          if (matchingImage) {
            stop.imageId = matchingImage.value;

            if (stop.imageId && !stop.imageId.startsWith('src')) {
              const apiUrlImage = `https://graph.mapillary.com/${stop.imageId}?access_token=${token}&fields=thumb_256_url&client_id=6817262911684044`;
              const response = await axios.get(apiUrlImage);
              stop.imageUrl = response.data.thumb_256_url;
            } else {
              stop.imageUrl = stop.imageId;
            }
          } else {
            stop.imageUrl = null;
          }
        } catch (error) {
          console.log("Error processing stop:", error);
          stop.imageUrl = null;
        }
      }
    },
    // Dessiner les circuits des lignes
    drawLines(transportLines){
      
      transportLines.forEach((transportLine) => {
      const coordinates = transportLine.shape.geometry.coordinates;
      coordinates.forEach((coords) => {
        coords.forEach((coord) => {
          coord.reverse();
        });
        L.polyline(coords, { color: `#${transportLine.route_color}`, stroke: true }).addTo(this.map);
      });
    });
    }
  },
};
</script>

<style scoped>
.map {
  height: 400px;
}
</style>
