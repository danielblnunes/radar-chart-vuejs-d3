<template>
  <div class="chart">
    <!-- FORM -->
    <div class="selectedTeams">
      <select
        v-model="selectedTeams"
        multiple="true"
        name="teams"
        id="teams"
        size="11"
      >
        <option v-for="team in teamsOutput" :value="team" :key="team">{{
          team
        }}</option>
      </select>
    </div>
    <svg :width="svgWidth" :height="svgHeight">
      <g
        class="wrapper"
        :transform="
          `translate(${width / 2 + margin.left}, ${height / 2 + margin.top})`
        "
      >
        <!-- CIRCLES -->
        <g v-for="circle in circleOutput" :key="circle.id">
          <circle
            class="circle"
            :cx="0"
            :cy="0"
            :r="circle.r"
            fill="grey"
            opacity="0.1"
          />
        </g>

        <!-- AXES -->
        <g class="axis" v-for="line in axisOutput" :key="line.id">
          <line
            class="line"
            :x1="line.x1"
            :y1="line.y1"
            :x2="line.x2"
            :y2="line.y2"
          />
        </g>

        <!-- AXES LABELS -->
        <g
          class="axisLabel"
          v-for="axisLabel in axisLabelOutput"
          :key="axisLabel.id"
        >
          <text :x="axisLabel.x" :y="axisLabel.y" dy="0.5em" class="axisLabel">
            {{ axisLabel.text }}
          </text>
        </g>

        <!-- CIRCLE LABELS -->
        <g
          class="circleLabel"
          v-for="label in labelOutput.slice(0, 4)"
          :key="label.id"
        >
          <text :x="label.x" :y="label.y" dy="0.5em" class="circleLabel">
            {{ label.text }}
          </text>
        </g>

        <!-- BLOBS -->
        <g class="radar" v-for="radar in radarOutput" :key="radar.id">
          <path
            class="radarArea"
            :d="radar.d"
            :fill="radar.fill"
            :stroke="radar.stroke"
            stroke-width="4px"
            :opacity="radar.opacity"
          />
          <path
            class="radarOutline"
            :d="radar.d"
            :fill="radar.fill"
            :stroke="radar.stroke"
          />
        </g>
        <!-- SMALL CIRCLES -->
        <g
          class="small-circles"
          v-for="scircle in smallCirclesOutput"
          :key="scircle.id"
        >
          <circle
            :cx="scircle.cx"
            :cy="scircle.cy"
            r="4"
            :fill="scircle.fill"
            opacity="0.8"
          />
        </g>

        <!-- LEGENG LABELS -->

        <g
          class="legendLabels textLegend"
          v-for="legend in legendLabelsOutput"
          :key="legend.id"
        >
          <circle
            @mouseover="hover = true"
            @mouseleave="hover = false"
            :cy="legend.y"
            cx="280"
            r="10"
            :fill="legend.fill"
          />
          <text
            :y="legend.textY"
            x="300"
            dy="0.3rem"
            font-size="16"
            class="textLegend"
          >
            {{ legend.text }}
          </text>
        </g>
      </g>
    </svg>
  </div>
</template>

<script>
import * as d3 from "d3";

const margin = { top: 100, right: 100, bottom: 100, left: 100 };

const width = 600 - margin.left - margin.right;
const height = 600 - margin.top - margin.bottom;

export default {
  name: "RadarGraph",
  props: ["motifData"],

  data() {
    return {
      hover: false,
      selectedTeams: [],
      height: height,
      width: width,
      svgWidth: width + margin.left + margin.right + 150,
      svgHeight: height + margin.top + margin.bottom,
      margin: margin,
    };
  },

  created() {},

  watch: {},

  mounted() {},

  methods: {
    renderAxis() {
      return {
        axisMaxValue: d3.max(this.motifData, (d) =>
          d3.max(
            Object.entries(d)
              .slice(1, 6)
              .map((o) => o[1])
          )
        ),
        axisNames: ["ABCB", "ABAC", "ABCD", "ABCA", "ABAB"],
        axisNumber: 5,
        radius: Math.min(this.width / 2, this.height / 2),
        format: d3.format(".0%"),
        angleSlice: (Math.PI * 2) / 5,
      };
    },

    calculateCircles() {
      const output = d3.range(0, 5).reverse();

      return output.map((d, i) => {
        return {
          id: i + 1,
          r: (this.width / 2 / 4) * d,
        };
      });
    },

    calculateAxes() {
      //Scale for the radius
      const rScale = d3
        .scaleLinear()
        .range([0, this.allAxis.radius])
        .domain([0, this.allAxis.axisMaxValue]);

      const axisRange = d3.range(0, this.allAxis.axisNumber);

      return axisRange.map((d, i) => {
        return {
          id: "axis-" + i,
          x1: 0,
          y1: 0,
          x2:
            rScale(this.allAxis.axisMaxValue * 1.05) *
            Math.cos(this.allAxis.angleSlice * i - Math.PI / 2),
          y2:
            rScale(this.allAxis.axisMaxValue * 1.05) *
            Math.sin(this.allAxis.angleSlice * i - Math.PI / 2),
        };
      });
    },

    calculateLabels() {
      const labels = d3.range(0, 5).reverse();

      return labels.map((d, i) => {
        return {
          id: "label-" + i,
          x: 7,
          y: (-d * this.allAxis.radius) / 4 - 6,
          text: this.allAxis.format((this.allAxis.axisMaxValue * d) / 4),
        };
      });
    },

    calculateAxisLabels() {
      //Scale for the radius
      const rScale = d3
        .scaleLinear()
        .range([0, this.allAxis.radius])
        .domain([0, this.allAxis.axisMaxValue]);

      const axisLabels = d3.range(0, this.allAxis.axisNumber);

      return axisLabels.map((d, i) => {
        return {
          id: "axisLabel-" + i,
          x:
            rScale(this.allAxis.axisMaxValue * 1.2) *
            Math.cos(this.allAxis.angleSlice * i - Math.PI / 2),
          y:
            rScale(this.allAxis.axisMaxValue * 1.2) *
            Math.sin(this.allAxis.angleSlice * i - Math.PI / 2),
          text: this.allAxis.axisNames.slice(i, i + 1)[0],
        };
      });
    },

    calculateRadar() {
      //Scale for the radius
      const rScale = d3
        .scaleLinear()
        .range([0, this.allAxis.radius])
        .domain([0, this.allAxis.axisMaxValue]);

      const colourScale = d3.scaleOrdinal(d3.schemeTableau10);

      const radialScale = d3
        .lineRadial()
        .curve(d3.curveCardinalClosed.tension(0.25));

      return this.computedData.map((d, i) => {
        const values = this.allAxis.axisNames.map((o) => rScale(d[o]));

        const points = values.map((d, i) => [i * this.allAxis.angleSlice, d]);

        return {
          id: "radar-" + i,
          d: radialScale(points),
          fill: colourScale(i),
          stroke: colourScale(i),
          opacity: this.hover === false ? 0.4 : 1,
        };
      });
    },

    calculateSmallCircles() {
      //Scale for the radius
      const rScale = d3
        .scaleLinear()
        .range([0, this.allAxis.radius])
        .domain([0, this.allAxis.axisMaxValue]);

      const colourScale = d3.scaleOrdinal(d3.schemeTableau10);

      const values = this.computedData.map((d, i) => {
        const points = this.allAxis.axisNames.map((q) => rScale(d[q]));

        const team = d["Team"].split(" ")[0];

        return points.map((o, e) => {
          return {
            id: "small-circle-" + i + "-" + e,
            cx: o * Math.cos(this.allAxis.angleSlice * e - Math.PI / 2),
            cy: o * Math.sin(this.allAxis.angleSlice * e - Math.PI / 2),
            fill: colourScale(i),
            team: team,
          };
        });
      });

      return values.flat();
    },
    calculateLegendLabels() {
      const colourScale = d3.scaleOrdinal(d3.schemeTableau10);

      return this.computedData.map((d, i) => {
        const team = d["Team"].split(" ")[0];

        const size = 20;

        const y = -200 + i * (size + 20);

        const textY = y;

        const fill = colourScale(i);

        return {
          id: "legend-" + i,
          text: team,
          y: y,
          textY: textY,
          fill: fill,
        };
      });
    },

    calculateTeams() {
      return this.motifData.map((d) => {
        const teams = d["Team"].split(" ")[0];

        return teams;
      });
    },
  },

  computed: {
    computedData() {
      const filterTeam = this.selectedTeams;

      const values = filterTeam.map((d) => {
        return this.motifData.filter((o) => o.Team.split(" ")[0] == d);
      });

      return values.flat();
    },

    teamsOutput() {
      return this.calculateTeams();
    },

    allAxis() {
      return this.renderAxis();
    },

    circleOutput() {
      return this.calculateCircles();
    },

    axisOutput() {
      return this.calculateAxes();
    },

    labelOutput() {
      return this.calculateLabels();
    },

    axisLabelOutput() {
      return this.calculateAxisLabels();
    },

    radarOutput() {
      return this.calculateRadar();
    },

    smallCirclesOutput() {
      return this.calculateSmallCircles();
    },

    legendLabelsOutput() {
      return this.calculateLegendLabels();
    },
  },
};
</script>

<style lang="scss">
/*RADAR CHART*/

.chart {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
}

svg {
  border: 0px solid;
  border-color: #ededed;
}

.circle {
  stroke: black;
  stroke-width: 2px;
}

.line {
  stroke: #ccc;
  stroke-width: 3;
}

.circleLabel {
  font-size: 10px;
}

.axisLabel {
  font-size: 11px;
  text-anchor: middle;
}

.radarArea {
  opacity: 0.4;
}

.radarHoveredArea {
  opacity: 1;
}

.radarOutline {
  stroke-width: 2;
  fill: none;
}

select {
  background-color: #ededed;
  color: #d64c29;
  border-color: #ededed;
  border-width: 1px;
  width: 100%;
  text-align: center;
}

.selectedTeams {
  margin: 1rem 0;
}

text {
  fill: #191919;
}

.text-legend {
  font-size: 1em;
}

@media (min-width: 600px) {
  .chart {
    display: flex;
    flex-direction: row;
    justify-content: center;
  }
}
</style>
