<template>
  <div id="app">
    <!-- This is only a basic setup to display your views. Use HTML and CSS to create your layout using these basic commands -->
    <ScatterPlot
      class="scatter-plot"
      v-if="dataset"
      chartId="poke1"
      title="All Fire Pokemons"
      :dataset="dataset"
      attribX="relStrength"
      labelX="Strength"
      attribY="relCatchRate"
      labelY="Catch Rate"
      attribColor="Name"
      attribGlyph="isLegendary"
      attribGlyphValNormal="False"
      attribGlyphValSpecial="True"
      @item-selected="pokemonSelected"
      :width="800"
      :height="500"
    />
    <BarChart 
      class="bar-chart" 
      v-if="datasetSelected" 
      chartId="poke2" 
      :title="`Health and attack about ${datasetSelected.name}`" 
      :dataset="datasetSelected.entries" 
      attribX="name" 
      attribY="value"  
      labelY="Strength"
      @item-selected="barSelected"
      :width="800"
      :height="500" 
    />
    <Bar2 
      class="bar-2" 
      v-if="datasetSelected" 
      chartId="poke3" 
      :title="`More info about ${datasetSelected.name}`" 
      :dataset="datasetSelected.entries2" 
      attribX="name" 
      attribY="value"  
      labelY="Strength"
      :width="800"
      :height="500" />
    <h1 v-else class="bar-chart-info">Select a Pokemon</h1>
  </div>
</template>

<script>
import * as d3 from "d3";

import ScatterPlot from "./components/ScatterPlot.vue";
import BarChart from "./components/BarChart.vue";
import Bar2 from "./components/Bar2.vue";

export default {
  name: "App",
  components: {
    ScatterPlot,
    BarChart,
    Bar2,
  },

  data() {
    return {
      dataset: null,
      datasetSelected: null,
      barselected:null,
    };
  },

  created() {
    d3.csv("/data/pokemon.csv", d3.autoType).then((data) => {
        let fire = d3.filter(data, d => d.Type_1 == "Fire")
        this.dataset = d3.map(fire, (d) => {
        return Object.assign(d, {
          relStrength: d.Total, 
          relCatchRate: d.Catch_Rate 
        })
      })
    });
  },
  methods: {
    pokemonSelected(p) {
      if (p == null) {
        this.datasetSelected = null
        return
      }
      let pokemon = d3.filter(this.dataset, d => d.Number == p.Number)[0]
      
      let entries = []
      let entries2 = []

      for (const stat of ['HP', 'Attack']) {
        entries.push({
          name: stat,
          value: pokemon[stat]
        })
      }
      for (const stat of ['Defense', 'Speed', 'Total']) {
        entries2.push({
          name: stat,
          value: pokemon[stat]
        })
      }
      this.datasetSelected = {
        name: pokemon.Name,
        pok:pokemon,
        entries: entries,
        entries2: entries2
      }
    },
  barSelected(q){
      if(q == null) {
        this.barselected = null
        return
      }
      if (q == this.datasetSelected.pok.HP){
        this.barselected = {
          name: this.datasetSelected.name,
          health: this.datasetSelected.pok.HP
        }
      }
    },
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: hsl(210, 29%, 24%);
  display: grid;
  grid-template-rows: 5vh 60vh auto;
  grid-template-columns: 1fr 10fr 10fr 1fr;
  grid-template-areas: 
    ". . . ." 
    ". overview detail ."
    ". overview detail2 ."
    ". . . .";
}
.scatter-plot {
  grid-area: overview;
}
.bar-chart {
  grid-area: detail;
}
.bar-2 {
  grid-area: detail2 ;
}
.bar-chart-info {
  grid-area: detail;
  color: rgb(214, 214, 214);
  margin: auto;
}
</style>