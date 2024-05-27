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
      <div class="table-responsive">
        <table v-if="jsonData.length">
          <thead>
            <tr>
              <th v-for="key in headers" :key="key" :class="{ 'text-left': key === 'Comune', 'text-right': key !== 'Comune' }">{{ key }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(row, index) in displayedData" :key="index">
              <td v-for="key in headers" :key="key" :class="{ 'text-left': key === 'Comune', 'text-right': key !== 'Comune' }" @click="handleCellClick(row[key], key)" :data-label="key">
                <button v-if="key === 'Comune'">{{ row[key] }}</button>
                <span v-else>{{ row[key] }}</span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

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
  background-color: #fff; /* Sfondo bianco */
}

.table-responsive {
  overflow-x: auto; /* Permette lo scorrimento orizzontale su dispositivi mobili */
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px; /* Spazio sopra la tabella */
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); /* Ombra leggera per la tabella */
}

th, td {
  border: 1px solid #ddd;
  padding: 12px; /* Aumenta il padding per una migliore leggibilità */
}

th {
  background-color: #f4f4f4;
  font-weight: bold;
  text-align: left;
}

tr:nth-child(even) {
  background-color: #f9f9f9; /* Colore di sfondo alternato per le righe */
}

tr:hover {
  background-color: #f1f1f1; /* Colore di sfondo al passaggio del mouse */
}

th, td {
  border: 1px solid #ddd; /* Colore dei bordi */
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
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  max-width: 80%;
  max-height: 80%;
  overflow-y: auto;
}

/* Stili per rendere la tabella responsive */
@media screen and (max-width: 768px) {
  table, thead, tbody, th, td, tr {
    display: block; /* Rendi la tabella block per dispositivi mobili */
  }
  thead tr {
    position: absolute;
    top: -9999px;
    left: -9999px;
  }
  tr {
    margin: 0 0 1rem 0;
  }
  tr:nth-child(even) {
    background: none;
  }
  td {
    border: none;
    border-bottom: 1px solid #ddd;
    position: relative;
    padding-left: 50%;
    text-align: right;
  }
  td:before {
    position: absolute;
    top: 50%;
    left: 10px;
    width: calc(50% - 20px);
    padding-right: 10px;
    white-space: nowrap;
    transform: translateY(-50%);
    text-align: left;
    font-weight: bold;
    content: attr(data-label); /* Utilizza l'attributo data-label per il contenuto */
  }
}
</style>
