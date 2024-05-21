<template>
  <div class="container">
    <h1>TEMPERATURE</h1>

    <!-- Inserimento del grafico -->
    <div>
      <apexchart type="line" height="350" :options="chartOptions" :series="series"></apexchart>
    </div>

    <!-- Tabella dei dati -->
    <table v-if="jsonData.length">
      <thead>
        <tr>
          <th v-for="key in headers" :key="key" :class="{ 'text-left': key === 'Anno' || key === 'Città', 'text-right': key !== 'Anno' && key !== 'Città' }">{{ key }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in jsonData" :key="index">
          <td v-for="(value, key) in row" :key="key" :class="{ 'text-left': key === 'Anno' || key === 'Città', 'text-right': key !== 'Anno' && key !== 'Città' }" @click="handleCellClick(value, key)">
            <button v-if="key === 'Città'">{{ value }}</button>
            <span v-else>{{ value }}</span>
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
      chartOptions: {
        chart: {
          id: 'temperature-chart'
        },
        xaxis: {
          categories: []
        },
        yaxis: {
          title: {
            text: 'Media Temperatura Annuo'
          }
        }
      },
      series: [],
      selectedCell: null
    };
  },
  mounted() {
    this.loadExcelFile();
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

        // Set x-axis categories for the chart
        this.chartOptions.xaxis.categories = this.headers.slice(2); // Assuming first two headers are 'Anno' and 'Città'

        // Generate series data for the chart
        this.series = this.jsonData.map(row => ({
          name: row.Città,
          data: this.headers.slice(2).map(year => row[year])
        }));

      } catch (error) {
        console.error('Errore durante il caricamento del file Excel:', error);
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
.container {
  margin: 0 5%;
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
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  max-width: 80%;
  max-height: 80%;
  overflow-y: auto;
}
</style>
