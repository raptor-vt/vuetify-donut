/** So I need to call adbd get the latest time fir a thing thtat I have */

<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12"> </v-col>
           <p>Slider Value: {{ sliderValue }}</p>
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <DonutChart :sliderValue="sliderValue" :centerCircleState="currentStateIndex"/>
      </v-col>
    </v-row>


    <v-row>
      <v-col>
        <v-btn @click="iterateMoleState()">Change Center Circle Color</v-btn>

      </v-col>
    </v-row>    

    <v-row>
      <v-col>
               <!--  step="0.01" Allows fine-grained control for decimal values -->
        <v-slider
          v-model="sliderValue"
          step="0.01"
          :min="0"
          :max="100"
          
          label="Slider"
        ></v-slider>

      </v-col>
    </v-row>        


  </v-container>
</template>

<script>
// Import the DonutChart component
import DonutChart from "@/components/DonutChart.vue"; // Adjust the path as necessary

export default {
  name: "Mole",
  components: {
    DonutChart,
  },
  data() {
    return {
      currentStateIndex: 0,
      sliderValue: 35.67, // Initial decimal value
    };
  },  
  methods: {

    iterateMoleState() {
      this.currentStateIndex =
        (this.currentStateIndex + 1) % 3;


    },

  }
};
</script>

<style></style>
