<template>
  <div id="app">
    <RadarChart :motifData="loadData" />
  </div>
</template>

<script>
import RadarChart from "./components/RadarChart.vue";
import * as d3 from "d3";

export default {
  name: "App",
  components: {
    RadarChart,
  },

  data() {
    return {
      loadData: [],
    };
  },
  mounted() {
    this.fetchData();
  },

  methods: {
    async fetchData() {
      let data = await d3.json("./team_motifs.json");
      this.loadData = data;
    },
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
