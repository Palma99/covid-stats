<template>
  <nav class="navbar navbar-expand navbar-dark bg-dark">
    <a class="navbar-brand" href="#">Covid Stats</a>
    <button
      class="navbar-toggler"
      type="button"
      data-toggle="collapse"
      data-target="#navbarColor01"
      aria-controls="navbarColor01"
      aria-expanded="false"
      aria-label="Toggle navigation"
    >
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navbarColor01">
      <ul class="navbar-nav mr-auto"></ul>
      <form class="form-inline" @submit.prevent="getData">
        <input
          class="form-control mr-sm-2"
          type="search"
          placeholder="Search"
          aria-label="Search"
          name="city"
          v-model="city"
        />
        <button class="btn btn-outline-info my-2 my-sm-0" type="submit">
          Get Covid
        </button>
      </form>
    </div>
  </nav>
  <div v-if="status" class="container mt-3">
    <h1>Statistiche {{ today.name }}</h1>
    <div class="card-group mt-4" style="justify-content: center;">
      <div class="card text-white bg-primary mb-3" style="max-width: 18rem;">
        <div class="card-header">Morti</div>
        <div class="card-body">
          <h5 class="card-title">Morti Totali</h5>
          <p class="card-text">{{ today.deaths }}</p>
        </div>
      </div>
      <div class="card text-white bg-info mb-3" style="max-width: 18rem;">
        <div class="card-header">Confermati</div>
        <div class="card-body">
          <h5 class="card-title">Confermati Totali</h5>
          <p class="card-text">{{ today.confirmed }}</p>
        </div>
      </div>
      <div class="card text-white bg-success mb-3" style="max-width: 18rem;">
        <div class="card-header">Guariti</div>
        <div class="card-body">
          <h5 class="card-title">Guariti Totali</h5>
          <p class="card-text">{{ today.recovered }}</p>
        </div>
      </div>
      <div class="card text-dark bg-warning mb-3" style="max-width: 18rem;">
        <div class="card-header">Percentuale morti</div>
        <div class="card-body">
          <h5 class="card-title">% morti</h5>
          <p class="card-text">{{ today.d_rate }}%</p>
        </div>
      </div>
      <div class="card text-white bg-danger mb-3" style="max-width: 18rem;">
        <div class="card-header">Casi per milione</div>
        <div class="card-body">
          <h5 class="card-title">Casi positivi per milione</h5>
          <p class="card-text">{{ today.per_milion }}</p>
        </div>
      </div>
      <div class="card text-white bg-dark mb-3" style="max-width: 18rem;">
        <div class="card-header">Popolazione</div>
        <div class="card-body">
          <h5 class="card-title">Popolazione</h5>
          <p class="card-text">{{ today.population }}</p>
        </div>
      </div>
    </div>
    <div class="mt-3">Aggiornato: {{ today.date }} {{ today.time }}</div>
  </div>
  <div class="container mt-5">
    <canvas id="chart"></canvas>
  </div>
</template>

<script>
import { ref } from "vue";
import Chart from "chart.js";
import fetch from "cross-fetch";

export default {
  setup() {
    let status = ref(false);
    const city = ref("");
    const today = ref({
      deaths: 0,
      confirmed: 0
    });

    let deathsSet = [];
    let confirmedSet = [];
    let recoveredSet = [];
    let labels = [];
    let chart;
    const API_URL_TODAY = "https://corona-api.com/countries";
    const API_URL_ALL = "https://api.covid19api.com/dayone/country/";
    async function getData() {
      try {
        const response = await fetch(API_URL_TODAY, {
          method: "GET",
          redirect: "follow"
        });
        let todayData = await response.json();
        todayData = todayData.data.filter(
          obj => obj.name.toLowerCase() === city.value.toLowerCase()
        );

        const totalResponse = await fetch(
          API_URL_ALL + city.value.toLowerCase(),
          {
            method: "GET",
            redirect: "follow"
          }
        );

        const allData = await totalResponse.json();

        deathsSet = [];
        confirmedSet = [];
        recoveredSet = [];
        labels = [];

        allData.forEach((obj, i) => {
          let lab = " ";
          deathsSet.push(obj.Deaths);
          confirmedSet.push(obj.Confirmed);
          recoveredSet.push(obj.Recovered);
          if (i % 10 === 0) {
            lab = obj.Date.split("T")[0]
              .split("-")
              .reverse()
              .join("-");
          }
          labels.push(lab);
        });

        // console.log(allData);
        if (todayData.length === 0) {
          status.value = false;
          console.log("Paese non valido");
        } else {
          // console.log(todayData[0].today);
          status.value = true;
          let updated = todayData[0].updated_at;
          let [date, time] = updated.split("T");
          date = date
            .split("-")
            .reverse()
            .join("/");
          time = time.split(".")[0];
          today.value = {
            deaths: todayData[0].latest_data.deaths,
            confirmed: todayData[0].latest_data.confirmed,
            recovered:
              todayData[0].latest_data.recovered |
              recoveredSet[recoveredSet.length - 1],
            name: todayData[0].name,
            population: todayData[0].population,
            per_milion:
              todayData[0].latest_data.calculated.cases_per_million_population,
            d_rate: (
              "" + todayData[0].latest_data.calculated.death_rate
            ).substr(0, 4),
            date,
            time
          };
        }

        const ctx = document.getElementById("chart").getContext("2d");
        if (chart) {
          chart.destroy();
        }
        chart = new Chart(ctx, {
          type: "line",
          data: {
            labels,
            datasets: [
              {
                label: "Morti",
                backgroundColor: "rgb(0, 123, 255, 0.5)",
                borderColor: "rgb(0, 123, 255)",
                pointRadius: 0,
                data: deathsSet
              },
              {
                label: "Confermati",
                backgroundColor: "rgb(23, 162, 184, 0.5)",
                borderColor: "rgb(23, 162, 184)",
                pointRadius: 0,
                data: confirmedSet
              },
              {
                label: "Guariti",
                backgroundColor: "rgb(40, 167, 69, 0.5)",
                borderColor: "rgb(40, 167, 69)",
                pointRadius: 0,
                data: recoveredSet
              }
            ]
          },
          options: {
            showXLabels: 10
          }
        });
      } catch (error) {
        console.error(error);
      }
    }
    return {
      getData,
      city,
      today,
      status,
      chart
    };
  }
};
</script>

<style lang="scss"></style>
