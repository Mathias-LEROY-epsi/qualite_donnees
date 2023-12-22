<template>
  <div class="container legend">
    <div class="row">
      <div class="col-4 inline">
        <span class="legend-icon"><img src="src/assets/bus.png" alt="icon de bus" /></span> Bus
      </div>
      <div class="col-4 inline">
        <span class="legend-icon"><img src="src/assets/tram.png" alt="icon de tram" /></span>
        Tramway
      </div>
      <div class="col-4 inline ps-4"><span class="legend-icon path-icon"></span> Trajet</div>
    </div>
  </div>
</template>

<script lang="ts">
export default {
  name: 'LegendComponent'
}
</script>

<style scoped>
.inline {
  display: inline-flex;
  align-items: center;
  padding-block: 0.5em;
}
.legend {
  background-color: white;
  border: 1px solid #ccc;
  padding: 10px;
  margin-top: 10px;
}

.legend h2 {
  font-size: 1.5em;
}

.legend-icon {
  display: inline-block;
  width: 20px;
  height: 20px;
  margin-right: 5px;
}
.path-icon {
  background-color: black;
  height: 0;
  border: 1px solid black;
}

ul {
  list-style-type: none;
}
</style>
