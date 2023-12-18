<template>
  <div>
    <div class="greetings">
    <h1 class="Black">Carte des transports en commun de la ville de Nantes</h1>
    </div>
    <MapComponent />
    <LegendComponent/>
  </div>
</template>

<script>
import MapComponent from "@/components/MapComponent.vue";
import LegendComponent from "@/components/LegendComponent.vue"

export default {
  components: {
      MapComponent,
      LegendComponent,
  },
};
</script>
<style>
.greetings h1 {
    font-family:Verdana, Geneva, Tahoma, sans-serif;
    font-weight: 500;
    text-align: center;
    margin: 1em;
    font-size: 2.5em;
}
</style>
