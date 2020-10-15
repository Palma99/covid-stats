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
          <h5 class="card-title">Morti giornalieri</h5>
          <p class="card-text">{{ today.deaths }}</p>
        </div>
      </div>
      <div class="card text-white bg-danger mb-3" style="max-width: 18rem;">
        <div class="card-header">Confermati</div>
        <div class="card-body">
          <h5 class="card-title">Confermati giornalieri</h5>
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
      <div class="card text-light bg-info mb-3" style="max-width: 18rem;">
        <div class="card-header">Positivi totali</div>
        <div class="card-body">
          <h5 class="card-title">Casi totali</h5>
          <p class="card-text">{{ today.totalCases }}</p>
        </div>
      </div>
      <!-- <div class="card text-white bg-dark mb-3" style="max-width: 18rem;">
        <div class="card-header">Tamponi</div>
        <div class="card-body">
          <h5 class="card-title">Tamponi Totali</h5>
          <p class="card-text">{{ today.totalTests }}</p>
        </div>
      </div> -->
    </div>
    <div class="container">
      <div class="row">
        <div class="col-sm-4">
          <div class="card">
            <div class="card-body">
              <h5 class="card-title">Record contagi</h5>
              <p class="card-text" style="color: var(--danger);font-size: 2em">
                {{ records.confirmed.value }}
              </p>
              <div>{{ records.confirmed.date }}</div>
            </div>
          </div>
        </div>
        <div class="col-sm-4">
          <div class="card">
            <div class="card-body">
              <h5 class="card-title">Record morti</h5>
              <p class="card-text" style="color: var(--primary);font-size: 2em">
                {{ records.deaths.value }}
              </p>
              <div>{{ records.deaths.date }}</div>
            </div>
          </div>
        </div>
        <div class="col-sm-4">
          <div class="card">
            <div class="card-body">
              <h5 class="card-title">Tamponi totali</h5>
              <p class="card-text" style="color: var(--dark);font-size: 2em">
                {{ today.totalTests }}
              </p>
              <div>{{ getTodayDate() }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- <div class="mt-3">Aggiornato: {{ today.date }} {{ today.time }}</div> -->
  </div>
  <div class="container mt-5">
    <div>
      <h1>Statistiche totali</h1>
      <canvas id="chart"></canvas>
    </div>
    <div class="container mt-5">
      <div>
        <h1>Statistiche giornaliere</h1>
        <canvas id="chart2"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
import { reactive, ref } from "vue";
import Chart from "chart.js";
import fetch from "cross-fetch";

export default {
  setup() {
    let status = ref(false);
    let records = reactive({
      deaths: {
        value: 0,
        date: ""
      },
      confirmed: {
        value: 0,
        date: ""
      }
    });
    const city = ref("");
    const today = ref({
      deaths: 0,
      confirmed: 0
    });

    let deathsSet = [];
    let confirmedSet = [];
    let recoveredSet = [];
    let dailyConfirmedSet = [0];
    let dailyDeathsSet = [0];

    let labels = [];
    let chart, chart2;

    const API_URL_TODAY = "https://coronavirus-19-api.herokuapp.com/countries";
    const API_URL_ALL = "https://api.covid19api.com/dayone/country/";
    async function getData() {
      try {
        const response = await fetch(API_URL_TODAY);
        let todayData = await response.json();
        todayData = todayData.filter(
          obj => obj.country.toLowerCase() === city.value.toLowerCase()
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

        dailyConfirmedSet = [0];
        dailyDeathsSet = [0];

        let prevConf, prevDeaths;

        let deathsRecord = {
          value: 0,
          date: ""
        };

        let confirmedRecord = {
          value: 0,
          date: ""
        };


        allData.forEach((obj, i) => {
          let lab = " ";

          if (i > 0) {
            const value = Math.abs(obj.Confirmed - prevConf);
            dailyConfirmedSet.push(value);
            if (value && value > confirmedRecord.value) {
              confirmedRecord.value = value;
              confirmedRecord.date = dateFormat(obj.Date);
            }

            const valueDeaths = Math.abs(obj.Deaths - prevDeaths);
            dailyDeathsSet.push(valueDeaths);
            if (valueDeaths && valueDeaths > deathsRecord.value) {
              deathsRecord.value = valueDeaths;
              deathsRecord.date = dateFormat(obj.Date);
            }

          }
          prevConf = obj.Confirmed;
          prevDeaths = obj.Deaths;

          deathsSet.push(obj.Deaths);
          confirmedSet.push(obj.Confirmed);
          recoveredSet.push(obj.Recovered);
          if (i % 1 === 0) {
            lab = dateFormat(obj.Date);
          }
          labels.push(lab);
        });

        records.deaths = deathsRecord;
        records.confirmed = confirmedRecord;

        // console.log(allData);
        if (todayData.length === 0) {
          status.value = false;
          console.log("Paese non valido");
        } else {
          status.value = true;
          today.value = {
            deaths: todayData[0].todayDeaths,
            confirmed: todayData[0].todayCases,
            recovered: todayData[0].recovered,
            name: todayData[0].country,
            totalCases: todayData[0].cases,
            deathsPerMilion: todayData[0].deathsPerOneMillion,
            totalTests: todayData[0].totalTests
          };
        }

        dailyConfirmedSet.push(today.value.confirmed);
        dailyDeathsSet.push(today.value.deaths);

        // Check if last item is the record
        if (dailyDeathsSet[dailyDeathsSet.length - 1] > deathsRecord.value) {
          deathsRecord.value = dailyDeathsSet[dailyDeathsSet.length - 1];
          deathsRecord.date = getTodayDate();
        }
        if (
          dailyConfirmedSet[dailyConfirmedSet.length - 1] >
          confirmedRecord.value
        ) {
          confirmedRecord.value =
            dailyConfirmedSet[dailyConfirmedSet.length - 1];
          confirmedRecord.date = getTodayDate();
        }

        labels.push(getTodayDate());

        const ctx2 = document.getElementById("chart2").getContext("2d");
        if (chart2) {
          chart2.destroy();
        }
        chart2 = new Chart(ctx2, {
          type: "line",
          data: {
            labels,
            datasets: [
              {
                label: "Positivi giornalieri",
                backgroundColor: "#dc354580",
                borderColor: "#dc3545",
                pointRadius: 0,
                data: dailyConfirmedSet
              },
              {
                label: "Morti giornalieri",
                backgroundColor: "#007bff88",
                borderColor: "#007bff",
                pointRadius: 0,
                data: dailyDeathsSet
              }
            ]
          },
          options: {
            showXLabels: 10
          }
        });
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

    const dateFormat = date => {
      return date
        .split("T")[0]
        .split("-")
        .reverse()
        .join("/");
    };

    const getTodayDate = () => {
      let today = new Date();
      const dd = String(today.getDate()).padStart(2, "0");
      const mm = String(today.getMonth() + 1).padStart(2, "0"); //January is 0!
      const yyyy = today.getFullYear();

      today = dd + "/" + mm + "/" + yyyy;
      return today;
    };

    return {
      getData,
      city,
      today,
      status,
      chart,
      chart2,
      records,
      getTodayDate
    };
  }
};
</script>

<style lang="scss"></style>
