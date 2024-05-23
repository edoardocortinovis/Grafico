<template>
    <div class="page-wrapper">
      <div class="container">
        <h1>TEMPERATURE</h1>
  
        <!-- Inserimento del grafico -->
        <div>
          <apexchart type="bar" height="350" :options="chartOptions" :series="chartSeries"></apexchart>
        </div>
  
        <br>
  
        <h1 align="center">Tabella</h1>
        <div style="text-align: right;">
          <input type="checkbox" style="margin-bottom: 20px;" v-model="filterHottest" @change="filterData">
          <label for="filterHottest">Mostra solo le 10 città più calde</label>
        </div>
        
        <!-- Tabella dei dati -->
        <table v-if="jsonData.length">
          <thead>
            <tr>
              <th v-for="key in headers" :key="key" :class="{ 'text-left': key === 'Comune', 'text-right': key !== 'Comune' }">{{ key }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(row, index) in displayedData" :key="index">
              <td v-for="key in headers" :key="key" :class="{ 'text-left': key === 'Comune', 'text-right': key !== 'Comune' }" @click="handleCellClick(row[key], key)">
                <button v-if="key === 'Comune'">{{ row[key] }}</button>
                <span v-else>{{ row[key] }}</span>
              </td>
            </tr>
          </tbody>
        </table>
  
        <!-- Overlay e Card delle informazioni -->
        <div class="overlay" v-if="selectedCell">
          <div class="card">
            <h2>Dettagli</h2>
            <p>{{ selectedCell }}</p>
            <button @click="closeCard">Chiudi</button>
          </div>
        </div>
      </div>
    </div>
  </template>

<script>
import * as XLSX from 'xlsx';
import VueApexCharts from 'vue3-apexcharts';

export default {
  name: 'HomeView',
  components: {
    apexchart: VueApexCharts
  },
  data() {
    return {
      jsonData: [],
      headers: [],
      filterHottest: false,
      chartOptions: {
        chart: {
          id: 'temperature-chart',
          height: 350,
          type: 'bar'
        },
        xaxis: {
          categories: []
        },
        yaxis: {
          title: {
            text: 'Temperatura (°C)'
          }
        },
        dataLabels: {
          enabled: false
        },
        legend: {
          position: 'bottom',
          formatter: (seriesName, opts) => {
            return this.chartSeries[opts.seriesIndex]?.name || seriesName;
          }
        },
        tooltip: {
          shared: true,
          intersect: false
        }
      },
      chartSeries: [{
        name: 'Temperature',
        data: []
      }],
      selectedCell: null,
      displayedData: []
    };
  },
  async mounted() {
    await this.loadExcelFile();
  },
  methods: {
    async loadExcelFile() {
      try {
        const response = await fetch(new URL('@/assets/temperature.xlsx', import.meta.url));
        const arrayBuffer = await response.arrayBuffer();
        const data = new Uint8Array(arrayBuffer);
        const workbook = XLSX.read(data, { type: 'array' });
        const firstSheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[firstSheetName];
        let json = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

        // Set the headers
        this.headers = json[0];
        json.shift(); // Remove the header row from data

        // Convert to object array
        this.jsonData = json.map(row => {
          let obj = {};
          this.headers.forEach((header, index) => {
            obj[header] = row[index];
          });
          return obj;
        });

        this.filterData();

      } catch (error) {
        console.error('Errore durante il caricamento del file Excel:', error);
      }
    },

    filterData() {
      if (this.filterHottest) {
        this.displayedData = this.jsonData
          .map(row => ({
            ...row,
            temperature: parseFloat(row[this.headers[1]])
          }))
          .sort((a, b) => b.temperature - a.temperature)
          .slice(0, 10);
        
        this.chartOptions.xaxis.categories = this.displayedData.map(row => row['Comune']);
        this.chartSeries[0].data = this.displayedData.map(row => row.temperature);
      } else {
        this.displayedData = this.jsonData;
        this.chartOptions.xaxis.categories = this.jsonData.map(row => row['Comune']);
        this.chartSeries[0].data = this.jsonData.map(row => parseFloat(row[this.headers[1]]));
      }
    },

    handleCellClick(value, key) {
      this.selectedCell = `${key}: ${value}`;
    },

    closeCard() {
      this.selectedCell = null;
    }
  }
};
</script>


<style scoped>
.page-wrapper {
  margin: 0 5%; /* Spazio dal 5% sui lati */
}

.container {
  border-radius: 10px; /* Bordi arrotondati */
  padding: 20px; /* Spazio interno */
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); /* Ombra leggera */
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid;
  padding: 8px;
}

th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.text-left {
  text-align: left;
}

.text-right {
  text-align: right;
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.card {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 1);
  max-width: 80%;
  max-height: 80%;
  overflow-y: auto;
}
</style>
