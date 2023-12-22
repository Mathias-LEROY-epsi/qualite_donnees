<template>
  <div>
    <div class="greetings">
      <h1 class="Black">Carte des transports en commun de la ville de Nantes</h1>
    </div>
    <div class="container">
      <DashboardComponent />
    </div>
  </div>
</template>

<script>
import DashboardComponent from '@/components/DashboardComponent.vue'

export default {
  components: {
    DashboardComponent
  }
}
</script>
<style>
.greetings h1 {
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  font-weight: 500;
  text-align: center;
  margin: 1em;
  font-size: 1.5em;
}

.container {
  display: flex;
  justify-content: center;
  align-items: start;
}
</style>
