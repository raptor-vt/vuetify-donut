<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <img src="../assets/logo.svg" class="my-3" contain height="200" />
      </v-col>

      <v-col class="mb-4">
        <h1 class="text-h3 font-weight-bold mb-3">Vehicle</h1>

        <p class="subheading font-weight-regular">
          Installed on, For help and collaboration with other Vuetify
          developers,
          <br />please join our online

          <a href="https://community.vuetifyjs.com" target="_blank"
            >Discord Community</a
          >
        </p>
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <DonutChart />
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
// Import the DonutChart component
import DonutChart from "@/components/DonutChart.vue"; // Adjust the path as necessary

export default {
  name: "Home",
  components: {
    DonutChart,
  },
};
</script>

<style></style>
