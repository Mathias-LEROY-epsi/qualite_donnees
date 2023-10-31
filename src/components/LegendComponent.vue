<template>
    <div class="container legend">
      <h2>Légende</h2>
      <div class="row">
        <div class="col-6">
            <ul>
                <li>
                    <span class="legend-icon bus-icon"></span> Bus
                </li>
                <li>
                    <span class="legend-icon tram-icon"></span> Tramway
                </li>
            </ul>
        </div>
        <div class="col-6">
            <ul>
                <li>
                    <span class="legend-icon path-icon"></span> Trajet
                </li>
            </ul>
        </div>
      </div>
    </div>
</template>
  
<script lang="ts">
export default {
name: "LegendComponent",
};
</script>

<style scoped>
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

.bus-icon {
background-color: #4472c4;
}

.tram-icon {
background-color: green;
}
.path-icon {
  background-color: black;
  height: 0; /* Largeur de 0 pour créer une ligne droite */
  border: 1px solid black; /* Ajoute une bordure pour créer la ligne */
}

ul {
    list-style-type: none;
}
</style>
