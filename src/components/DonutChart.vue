<template>
  <v-container fluid fill-height class="chart-container">
    <!-- Flex box to center the chart vertically and horizontally -->
    <v-row align="start" justify="center" class="fill-width">
      <v-col cols="12">
        <Doughnut
          ref="doughnutChart"
          :chart-options="chartOptions"
          :chart-data="chartData"
          :chart-id="chartId"
          :dataset-id-key="datasetIdKey"
          :css-classes="cssClasses"
          :styles="styles"
          :width="width"
          :height="height"
          :plugins="[centerTextPlugin, centerCirclePlugin]"
        />
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <v-divider></v-divider>

        <v-btn @click="iterateMoleState()">Change Center Circle Color</v-btn>

        <v-divider></v-divider>
      </v-col>

      <v-col>
        <v-divider></v-divider>

        {{ internalSliderValue }}

        <v-divider></v-divider>
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <v-select
          v-model="legendPosition"
          :items="legendPositions"
          label="Legend Position"
          outlined
          dense
          @change="updateLegendPosition"
        >
        </v-select>

        <v-divider></v-divider>

        <!--  step="0.01" Allows fine-grained control for decimal values -->
        <v-slider
          v-model="internalSliderValue"
          step="0.01"
          :min="0"
          :max="100"
          @input="updateChart"
          label="Slider"
        ></v-slider>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
const MoleState = Object.freeze({
  ACTIVE: "ACTIVE",
  INACTIVE: "INACTIVE",
  REPLACE: "REPLACE",
});

const MoleStateArray = Object.values(MoleState);

import { Doughnut } from "vue-chartjs/legacy";

import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  ArcElement,
  CategoryScale,
} from "chart.js";

ChartJS.register(Title, Tooltip, Legend, ArcElement, CategoryScale);

export default {
  name: "DoughnutChart",
  components: {
    Doughnut,
  },
  props: {
    chartId: {
      type: String,
      default: "doughnut-chart",
    },
    datasetIdKey: {
      type: String,
      default: "label",
    },
    width: {
      type: Number,
      default: 400,
    },
    height: {
      type: Number,
      default: 300,
    },
    cssClasses: {
      default: "",
      type: String,
    },
    styles: {
      type: Object,
      default: () => {},
    },
    /*     value: {
      type: Number,
      required: true,
    },
 */
    sliderValue: {
      type: Number,
      default: 0, // Default value for the slider is 0
    },
    centerCircleState: {
      // 0 - active; 1 - inactive ; 2 - replace
      type: Number,
      default: 0, // Default to red if no color is provided
    },
    /* plugins: {
      type: Array,
      default: () => [],
    }, */
  },
  data() {
    return {
      circleColors: ["#2196f3", "#ff9700", "#f44236"], // Array of colors for the center circle
      circleCaptions: ["Active", "Inactive", "Replace"], // array of text desriptions for center circle

      legendPosition: "left", // Default selected value
      legendPositions: [
        { text: "Top", value: "top" },
        { text: "Bottom", value: "bottom" },
        { text: "Left", value: "left" },
        { text: "Right", value: "right" },
      ], // Array of legend position options

      //legendPosition: "top", // Default legend position
      // internalSliderValue: this.sliderValue,
      // sliderValue: this.value, // Initialize internal value from prop, // Slider value ranging from 0 to 100

      currentColorIndex: 0, // Index to track the current color
      currentStateIndex: 0,
      chartData: {
        labels: ["Elapsed ", "Year 1", "Year 2", "Year 3"], // Start with gray section
        datasets: [
          // Second Dataset (Secondary Ring)
          /* {
            label: "Gray Section Expansion",
            data: [100, 0], // Gray section and empty space initially
            // backgroundColor: ["#808080", "rgba(0, 0, 0, 0)"], // Gray and transparent for empty space
            backgroundColor: ["#808080", "#ffffff"], // Gray and transparent for empty space
            borderWidth: 1,
            labels: ["Elapsed", "Remaining"], //
          }, */

          {
            label: "Main Segments",
            labels: ["Elapsed", "Year 1", "Year 2", "Year 3"], // Start with gray section
            data: [0, 33.3, 33.3, 33.3], // Initialize sections with 0 for the gray segment
            backgroundColor: ["#c0c0c0", "#2aa12a", "#fc7b0c", "#539dd5"], // Gray first, followed by colors
            borderWidth: 1,
          },

          {
            label: "Main Segments",
            labels: ["Year 1", "Year 2", "Year 3"], // Start with gray section
            data: [33.3, 33.3, 33.3], // Initialize sections with 0 for the gray segment
            backgroundColor: ["#2aa12a", "#fc7b0c", "#539dd5"], // Gray first, followed by colors
            borderWidth: 1,
          },
        ],
      },
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        cutout: "45%", // This creates a hole for the concentric rings
        circumference: 360, // Controls the circle's circumference
        /// rotation: -90, // Starts the drawing from the top (optional)
        percentage: this.sliderValue,

        // percentage: 12,
        circleColor: "#2196f3", // Initial color for the center circle
        circleCaption: "Active",

        plugins: {
          legend: {
            display: true,
            position: "left", // Set the default legend position
          },
        },

        /*         plugins: {
          legend: {
            display: true,
            position: "left",
            labels: {
              generateLabels(chart) {
                // Generate custom labels for each dataset
                const datasets = chart.data.datasets;
                let legends = [];

                datasets.forEach((dataset, datasetIndex) => {
                  dataset.data.forEach((value, index) => {
                    legends.push({
                      text: dataset.labels[index] || chart.data.labels[index], // Use dataset-specific labels
                      fillStyle: dataset.backgroundColor[index],
                      hidden: !chart.isDatasetVisible(datasetIndex),
                      datasetIndex,
                      index,
                    });
                  });
                });

                return legends;
              },
            },
          },
        }, */

        /*         plugins: {
          legend: {
            display: true,
            position: "left",
            labels: {
              generateLabels(chart) {
                // This will create custom labels for each dataset
                return chart.data.datasets.map((dataset, i) => ({
                  text: dataset.label,
                  fillStyle: dataset.backgroundColor[1], // Set the legend color to match dataset color
                  hidden: !chart.isDatasetVisible(i),
                  index: i,
                }));
              },
            },
          },
        }, */
      },

      centerCirclePlugin: {
        id: "centerCirclePlugin",
        beforeDraw(chart) {
          const ctx = chart.ctx;
          const centerX = (chart.chartArea.left + chart.chartArea.right) / 2;
          const centerY = (chart.chartArea.top + chart.chartArea.bottom) / 2;

          ctx.save();
          const radius = chart.getDatasetMeta(0).data[0].innerRadius * 0.6;
          ctx.beginPath();
          ctx.arc(centerX, centerY, radius, 0, 2 * Math.PI);
          ctx.fillStyle = chart.config.options.circleColor;
          ctx.fill();
          ctx.restore();
        },
      },

      centerTextPlugin: {
        id: "centerTextPlugin",
        afterDraw(chart) {
          const ctx = chart.ctx;
          const centerX = (chart.chartArea.left + chart.chartArea.right) / 2;
          const centerY = (chart.chartArea.top + chart.chartArea.bottom) / 2;

          const text = chart.config.options.circleCaption;

          ctx.save();
          ctx.font = "20px Arial";
          ctx.textAlign = "center";
          ctx.fillStyle = "#000";
          ctx.fillText(`${text}`, centerX, centerY);
        },
      },
    };
  },

  mounted() {
    this.changeMoleState(this.centerCircleState);
  },

  watch: {
    centerCircleState: {
      handler(newValue) {
        console.log("centerCircleState:" + newValue);
        const graySegmentSize = (this.internalSliderValue / 100) * 100; // Calculate the total size of the gray section

        if (graySegmentSize > 91.6) {
          this.changeMoleState(2);
        } else {
          this.changeMoleState(newValue);
        }

        this.updateChart();
      },
      immediate: true, // Run the watcher immediately to handle initial prop changes
    },

    // Watch for changes in sliderValue prop and update the chart
    sliderValue: {
      handler(newValue) {
        this.internalSliderValue = newValue;
        this.updateChart();
      },
      immediate: true, // Run the watcher immediately to handle initial prop changes
    },
    // Watch for changes in chartData and ensure the chart is re-rendered
    chartData: {
      deep: true, // Deep watch for nested changes in chartData
      handler() {
        // this.$refs.doughnutChart && this.$refs.doughnutChart.update(); // Manually trigger chart re-render
        // this.updateChart();
      },
    },
  },

  methods: {
    updateLegendPosition() {
      // Update the legend position in the chart options
      this.chartOptions.plugins.legend.position = this.legendPosition;
    },

    /**
     *  V 0.1.0 - 2024-09-12
     *
     */
    changeMoleState(index) {
      if (index >= 0 || index <= 2) {
        this.chartOptions.circleColor = this.circleColors[index];
        this.chartOptions.circleCaption = this.circleCaptions[index];
      }
    },

    /**
     *  V 0.1.0 - 2024-09-12
     *
     */
    changeMoleState2(state) {
      let index = 0;

      switch (state) {
        case MoleState.ACTIVE:
          index = 0;
          break;
        case MoleState.INACTIVE:
          index = 1;
          break;
        case MoleState.REPLACE:
          index = 2;
          break;
        default:
          console.log("Not a valid day.");
      }

      this.chartOptions.circleColor = this.circleColors[index];
      this.chartOptions.circleCaption = this.circleCaptions[index];
    },

    /**
     *
     *     iterate through for testing,
     *
     */
    iterateMoleState() {
      this.currentStateIndex =
        (this.currentStateIndex + 1) % MoleStateArray.length;

      this.chartOptions.circleColor = this.circleColors[this.currentStateIndex];
      this.chartOptions.circleCaption =
        this.circleCaptions[this.currentStateIndex];
    },

    changeCircleColor() {
      // Change the center circle color by cycling through the colors
      this.currentColorIndex =
        (this.currentColorIndex + 1) % this.circleColors.length;

      this.chartOptions.circleColor = this.circleColors[this.currentColorIndex];

      this.chartOptions.circleCaption =
        this.circleCaptions[this.currentColorIndex];
    },

    updateChart() {
      const graySegmentSize = (this.internalSliderValue / 100) * 100; // Calculate the total size of the gray section

      // First phase: the gray section expands into Section 1
      if (graySegmentSize <= 33.3) {
        this.chartData.datasets[0].data = [
          graySegmentSize, // Gray section grows
          33.3 - graySegmentSize, // Section 1 shrinking
          33.3, // Section 2 remains constant
          33.3, // Section 3 remains constant
        ];
      } else if (graySegmentSize <= 66.6) {
        // Second phase: the gray section expands into Section 3
        this.chartData.datasets[0].data = [
          graySegmentSize, // Gray section grows
          0, // Section 1 fully taken over
          33.3 - (graySegmentSize - 33.3), // Section 2 remains constant
          33.3, // Section 3 shrinking
        ];
      } else {
        // Final phase: fully consumed
        this.chartData.datasets[0].data = [
          graySegmentSize, // Gray section grows fully
          0, // Section 1 fully taken over
          0, // Section 2 remains constant
          33.3 - (graySegmentSize - 66.6),
        ];
      }

      //
      if (graySegmentSize > 91.6) {
        this.changeMoleState(2);
      } else {
        this.changeMoleState(this.centerCircleState);
      }
    },

    updateChart3() {
      const graySectionSize = (this.internalSliderValue / 100) * 100;
      this.chartOptions.percentage = this.internalSliderValue;

      // Update Second Dataset (Secondary Ring)
      this.chartData.datasets[0].data = [
        graySectionSize,
        100 - graySectionSize,
      ];
    },

    updateChart2() {
      const graySegmentSize = (this.internalSliderValue / 100) * 100; // Calculate the total size of the gray section

      // First phase: the gray section expands into Section 1
      if (graySegmentSize <= 33.3) {
        this.chartData.datasets[0].data = [
          graySegmentSize, // Gray section grows
          33.3 - graySegmentSize, // Section 1 shrinking
          33.3, // Section 2 remains constant
          33.3, // Section 3 remains constant
        ];
      } else if (graySegmentSize <= 66.6) {
        // Second phase: the gray section expands into Section 3
        this.chartData.datasets[0].data = [
          graySegmentSize, // Gray section grows
          0, // Section 1 fully taken over
          33.3 - (graySegmentSize - 33.3), // Section 2 remains constant
          33.3, // Section 3 shrinking
        ];
      } else {
        // Final phase: fully consumed
        this.chartData.datasets[0].data = [
          graySegmentSize, // Gray section grows fully
          0, // Section 1 fully taken over
          0, // Section 2 remains constant
          33.3 - (graySegmentSize - 66.6),
        ];
      }

      //
      if (graySegmentSize > 91.6) {
        this.changeMoleState(MoleState.REPLACE);
      } else {
        this.changeMoleState(MoleState.ACTIVE);
      }
    },
  },
};
</script>

<style scoped>
.chart-container {
  width: 100vw; /* Full width */
  height: auto; /* Chart will auto-size based on its content */
  display: flex;
  align-items: flex-start; /* Align the chart to the top */
  padding: 0;
}

.full-width {
  width: 100%; /* Ensure the chart row uses full width */
}

.v-slider {
  margin-top: 10px; /* Add some spacing for aesthetics */
}
</style>
