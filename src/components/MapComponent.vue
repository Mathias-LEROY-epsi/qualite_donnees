<template>
    <div ref="map" class="map"></div>
</template>
  
<script>
    import L from 'leaflet';
    import 'leaflet/dist/leaflet.css';
    import axios from 'axios';

    const apiUrlStops = "https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records";
    const apiUrlLines = "https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-circuits/records?where=route_type%20%3D%20%22Bus%22";

    export default {
        data() {
            return {
                busStops: [],
                busLines: [],
                map: null,
            };
        },
        async created() {
            try {
                const stops = await axios.get(apiUrlStops);
                this.busStops = stops.data.results;
                //console.log(this.busStops);
                const lines = await axios.get(apiUrlLines);
                this.busLines = lines.data.results;
                //console.log(this.busLines);
                this.initMap();
            } catch (error) {
                console.log(error);
            }
        },
        methods: {
            initMap() {
                this.map = L.map(this.$refs.map).setView([47.2184, -1.5536], 12);
        
                //console.log(this.busStops);
                this.busStops.forEach(stop => {
                    //console.log(stop.stop_coordinates.lat);
                    const marker = L.circleMarker([stop.stop_coordinates.lat, stop.stop_coordinates.lon], {
                        radius: 5,
                        fillColor: 'black',
                        color: 'red',
                        weight: 1,
                    }).addTo(this.map); //--> FONCTIONNE
                });

                this.busLines.forEach(line => {
                    //console.log(line.shape.geometry.coordinates);
                    const lineColor = line.route_color;
                    console.log(lineColor);
                    this.busLines.forEach(line => {
                        const coordinates = line.shape.geometry.coordinates.map(coord => [coord[1], coord[0]]); // Inversion des coordonnées
                        console.log(coordinates[0]);
                        L.polyline(coordinates, { color: 'blue' }).addTo(this.map); //--> NE FONCTIONNE PAS
                    });

                    var latlngs = [
                        [47.30897242, -1.54466143],
                        [47.30838791, -1.54329479]
                    ];
                    L.polyline(latlngs, {color: 'purple'}).addTo(this.map); //--> FONCTIONNE
                });
                L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                maxZoom: 19,
                }).addTo(this.map);

            },
        },
    };
</script>
  
<style scoped>
  .map {
    height: 400px;
  }
</style>