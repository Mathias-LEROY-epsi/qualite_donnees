<template>
    <div ref="map" class="map"></div>
  </template>
  
<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import axios from 'axios';

const apiUrlStops = "https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-arrets/records";
const apiUrlLines = "https://data.nantesmetropole.fr/api/explore/v2.1/catalog/datasets/244400404_tan-circuits/records?where=route_type%20LIKE%20%22Tram%22%20OR%20%22Bus%22&limit=100";

export default {
data() {
    return {
    busStops: [],
    tanLines: [],
    busLines: [],
    tramLines: [],
    map: null,
    };
},
async created() {
    try {
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

    this.initMap();
    } catch (error) {
    console.log(error);
    }
},
methods: {
    initMap() {
    this.map = L.map(this.$refs.map).setView([47.2184, -1.5536], 12);

    this.tramLines.forEach(tramLine => {
        const coordinates = tramLine.shape.geometry.coordinates;
        console.log(coordinates)
        const latLngs = coordinates.map(coordinate => L.latLng(coordinate[0][1], coordinate[0][0]));

        L.polyline(latLngs, { color: `#${tramLine.route_color}` }).addTo(this.map);
    });

    this.busLines.forEach(busLine => {
        const coordinates = busLine.shape.geometry.coordinates;
        const latLngs = coordinates.map(coordinate => L.latLng(coordinate[0][1], coordinate[0][0]));
        console.log(typeof busLine.route_color);

        L.polyline(latLngs, { color: `#${busLine.route_color}` }).addTo(this.map);
    });

    this.busStops.forEach(stop => {
        const marker = L.circleMarker([stop.stop_coordinates.lat, stop.stop_coordinates.lon], {
        radius: 5,
        fillColor: 'black',
        color: 'red',
        weight: 1,
        }).addTo(this.map);
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