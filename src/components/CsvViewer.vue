<template>
    <div>
      <input type="file" @change="loadFile" accept=".csv, .xlsx" />
      <div v-if="csvData.length">
        <select v-model="selectedServer">
          <option v-for="server in uniqueServers" :key="server" :value="server">
            {{ server }}
          </option>
        </select>
        <div v-if="filteredData.length">
          <select v-model="selectedPseudo">
            <option v-for="row in filteredData" :key="row[pseudoKey]" :value="row[pseudoKey]">
              {{ row[pseudoKey] }}
            </option>
          </select>
          <div v-if="selectedRow">
            <h2>Details of {{ selectedPseudo }}</h2>
            <ul>
              <li v-for="(value, key) in selectedRow" :key="key">
                {{ key }}: {{ value }}
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import Papa from 'papaparse';
  import * as XLSX from 'xlsx';
  
  export default {
    data() {
      return {
        csvData: [],
        selectedServer: null,
        selectedPseudo: null,
        pseudoKey: '',  // The pseudo column name (index)
        pseudoCompteKey: '',  // The pseudo compte column name
        serverKey: ''  // The server column name
      };
    },
    computed: {
      uniqueServers() {
        const servers = this.csvData.map(row => row[this.serverKey]);
        return [...new Set(servers)];
      },
      filteredData() {
        return this.csvData.filter(row => row[this.serverKey] === this.selectedServer);
      },
      selectedRow() {
        return this.filteredData.find(row => row[this.pseudoKey] === this.selectedPseudo) || null;
      }
    },
    methods: {
      loadFile(event) {
        const file = event.target.files[0];
        if (file) {
          const fileExtension = file.name.split('.').pop().toLowerCase();
          if (fileExtension === 'csv') {
            this.parseCsv(file);
          } else if (fileExtension === 'xlsx') {
            this.parseXlsx(file);
          } else {
            console.error('Unsupported file format');
          }
        }
      },
      parseCsv(file) {
        Papa.parse(file, {
          header: true,
          complete: (results) => {
            this.csvData = results.data;
            const headers = Object.keys(this.csvData[0]);
            this.pseudoKey = headers[0];  // Assuming the first column is the pseudo key
            this.pseudoCompteKey = headers[6];  // Assuming the 7th column is the pseudo compte key
            this.serverKey = headers[4];  // Assuming the 5th column is the server key
          },
          error: (error) => {
            console.error('Error parsing CSV:', error);
          }
        });
      },
      parseXlsx(file) {
        const reader = new FileReader();
        reader.onload = (event) => {
          const data = new Uint8Array(event.target.result);
          const workbook = XLSX.read(data, { type: 'array' });
          const firstSheetName = workbook.SheetNames[1];
          const worksheet = workbook.Sheets[firstSheetName];
          const csvData = XLSX.utils.sheet_to_csv(worksheet);
          this.convertCsvToJson(csvData);
        };
        reader.readAsArrayBuffer(file);
      },
      convertCsvToJson(csvData) {
        Papa.parse(csvData, {
          header: true,
          complete: (results) => {
            this.csvData = results.data;
            const headers = Object.keys(this.csvData[0]);
            this.pseudoKey = headers[0];  // Assuming the first column is the pseudo key
            this.pseudoCompteKey = headers[6];  // Assuming the 7th column is the pseudo compte key
            this.serverKey = headers[4];  // Assuming the 5th column is the server key
          },
          error: (error) => {
            console.error('Error parsing CSV:', error);
          }
        });
      }
    }
  };
  </script>
  