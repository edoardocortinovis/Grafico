<template>
<h1>PRECIPITAZIONI</h1>
  <div class="container">
    <!-- Inserimento del grafico sopra la tabella -->
    <div>
      <apexchart type="line" height="350" :options="chartOptions" :series="series"></apexchart>
    </div>

    <!-- Inserimento della tabella sotto il grafico -->
    <table v-if="jsonData.length">
      <thead>
        <tr>
          <th v-for="(value, key) in jsonData[0]" :key="key" :class="{ 'text-left': key === 'Comuni', 'text-right': key !== 'Comuni' }">{{ key }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in jsonData" :key="index">
          <td v-for="(value, key) in row" :key="key" :class="{ 'text-left': key === 'Comuni', 'text-right': key !== 'Comuni' }">{{ value }}</td>
        </tr>
      </tbody>
    </table>
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
      chartOptions: {
        chart: {
          id: 'vuechart-example'
        },
        xaxis: {
          categories: [2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017, 2018, 2019, 2020, 2021]
        },
        yaxis: [
          {
            title: {
              text: 'Temperatura Media Annuo (°C)'
            },
          },
          {
            opposite: true,
            title: {
              text: 'Precipitazione Totale Annuo (mm)'
            },
          },
        ],
        stroke: {
          curve: 'smooth'
        },
        tooltip: {
          shared: true,
          intersect: false
        }
      },
      series: [
        {
          name: 'Temperatura Media Annuo',
          type: 'line',
          data: [15.02, 15.43, 14.55, 15.09, 13.87, 15.37, 14.97, 14.41, 15.28, 14.79, 15.31, 14.67, 14.95, 15.2, 14.67, 14.56]
        },
        {
          name: 'Precipitazione Totale Annuo',
          type: 'line',
          data: [871.4, 1035.4, 1150.2, 894.2, 975.4, 485.8, 1080.8, 1203.4, 842.8, 860.8, 781.8, 585.7, 753.8, 934.2, 1376.6, 700.2]
        }
      ]
    }
  },
  mounted() {
    this.loadExcelFile();
  },
  methods: {
    async loadExcelFile() {
      try {
        const response = await fetch(new URL('@/assets/precipitazioni.xlsx', import.meta.url));
        const arrayBuffer = await response.arrayBuffer();
        const data = new Uint8Array(arrayBuffer);
        const workbook = XLSX.read(data, { type: 'array' });
        const firstSheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[firstSheetName];
        let json = XLSX.utils.sheet_to_json(worksheet);
       
        // Rimuovi intestazioni vuote
        json = json.filter(row => Object.values(row).some(cell => cell !== ''));
       
        this.jsonData = json;
       
        // Visualizza array di elementi nella console
        console.log('Array di elementi:', this.jsonData);
      } catch (error) {
        console.error('Error loading Excel file:', error);
      }
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
</style>
