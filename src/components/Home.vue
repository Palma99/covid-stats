<template>
  <div class="container mt-3">
    <h1>Statistiche gobali</h1>
    <div class="card-group mt-4" style="justify-content: center;">
      <div class="card text-white bg-primary mb-3" style="max-width: 18rem;">
        <div class="card-header">Morti Globali</div>
        <div class="card-body">
          <h5 class="card-title">Morti Totali</h5>
          <p class="card-text">{{ globalData.TotalDeaths }}</p>
        </div>
      </div>
      <div class="card text-white bg-danger mb-3" style="max-width: 18rem;">
        <div class="card-header">Confermati Globali</div>
        <div class="card-body">
          <h5 class="card-title">Confermati Totali</h5>
          <p class="card-text">{{ globalData.TotalConfirmed }}</p>
        </div>
      </div>
      <div class="card text-white bg-success mb-3" style="max-width: 18rem;">
        <div class="card-header">Guariti Globali</div>
        <div class="card-body">
          <h5 class="card-title">Guariti Totali</h5>
          <p class="card-text">{{ globalData.TotalRecovered }}</p>
        </div>
      </div>
    </div>

    <h1 class="mt-5">Statistiche giornaliere</h1>
    <div class="card-group mt-4" style="justify-content: center;">
      <div class="card text-white bg-primary mb-3" style="max-width: 18rem;">
        <div class="card-header">Morti Globali</div>
        <div class="card-body">
          <h5 class="card-title">Morti Giornalieri</h5>
          <p class="card-text">{{ globalData.NewDeaths }}</p>
        </div>
      </div>
      <div class="card text-white bg-danger mb-3" style="max-width: 18rem;">
        <div class="card-header">Confermati Globali</div>
        <div class="card-body">
          <h5 class="card-title">Confermati Giornalieri</h5>
          <p class="card-text">{{ globalData.NewConfirmed }}</p>
        </div>
      </div>
      <div class="card text-white bg-success mb-3" style="max-width: 18rem;">
        <div class="card-header">Guariti Globali</div>
        <div class="card-body">
          <h5 class="card-title">Guariti Giornalieri</h5>
          <p class="card-text">{{ globalData.NewRecovered }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import Chart from "chart.js";
import fetch from "cross-fetch";
import { ref } from "vue";

const API_URL = "https://api.covid19api.com/summary";
export default {
  setup() {
    let globalData = ref({
      NewConfirmed: 0,
      NewDeaths: 0,
      NewRecovered: 0,
      TotalConfirmed: 0,
      TotalDeaths: 0,
      TotalRecovered: 0
    });
    async function getGlobalData() {
      try {
        const response = await fetch(API_URL, {
          method: "GET",
          redirect: "follow"
        });
        const json = await response.json();
        globalData.value = { ...json.Global };
      } catch (error) {
        console.error(error);
      }
    }
    getGlobalData();

    return {
      globalData
    };
  }
};
</script>

<style></style>
