<template>
  <div class="full">
    <Header class="Header" />
    <div class="top">
      <div class="expected-date">
        <label for="chooseDate">Choose the Date</label>
        <input type="date" id="chooseDate" v-model="searchDate" @input="updatedate" class="custom-input" />
      </div>
      <div class="search-input">
        <span class="search-icon" @click="searchReport">&#128269;</span>
      </div>
    </div>
    <div class="primary-table">
      <div class="button-group">
        <button type="button" class="primary-btn" :class="{ 'selected': selectedTable === 'patientBooking' }"
          @click="toggleTable('patientBooking')">Patient Booking</button>
        <button type="button" class="secondary-btn" :class="{ 'selected': selectedTable === 'queriedPatient' }"
          @click="toggleTable('queriedPatient')">Queried Patient</button>
        <button type="button" class="third-btn" :class="{ 'selected': selectedTable === 'rejectedPatient' }"
          @click="toggleTable('rejectedPatient')">Rejected Patient</button>
      </div>
    </div>
    <div class="table-container" :class="{ 'selected-table': selectedTable === 'patientBooking' }">
      <table v-if="selectedTable === 'patientBooking'" class="table">
        <thead>
          <tr>
            <th>Phone Number</th>
            <th>Patient Choice</th>
            <th>Timestamp</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="paginatedItems.length === 0">
            <td colspan="3" class="no-data-message">No data available</td>
          </tr>
          <tr v-else v-for="(item, index) in paginatedItems" :key="index">
            <td data-title="phonenumber">{{ item.phone_number }}</td>
            <td data-title="patientchoice">{{ item.doctor_choice }}</td>
            <td data-title="timestamp">{{ item.timestamp }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Queried Patient Table -->
    <div class="table-container" :class="{ 'selected-table': selectedTable === 'queriedPatient' }">
      <table v-if="selectedTable === 'queriedPatient'" class="table">
        <thead>
          <tr>
            <th>Phone Number</th>
            <th>TimeStamp</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="paginatedItems.length === 0">
            <td colspan="2" class="no-data-message">No data available</td>
          </tr>
          <tr v-else v-for="(item, index) in paginatedItems" :key="index">
            <td data-title="phonenumber">{{ item.phone_number }}</td>
            <td data-title="timestamp">{{ item.timestamp }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Rejected Patient Table -->
    <div class="table-container" :class="{ 'selected-table': selectedTable === 'rejectedPatient' }">
      <table v-if="selectedTable === 'rejectedPatient'" class="table">
        <thead>
          <tr>
            <th>Phone Number</th>
            <th>TimeStamp</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="paginatedItems.length === 0">
            <td colspan="2" class="no-data-message">No data available</td>
          </tr>
          <tr v-else v-for="(item, index) in paginatedItems" :key="index">
            <td data-title="phonenumber">{{ item.phone_number }}</td>
            <td data-title="timestamp">{{ item.timestamp }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
  <nav aria-label="Page navigation" class="pagination-container">
    <ul class="pagination">
      <li class="page-item">
        <a class="page-link" @click="changePage(pagination.page - 1)" href="#" :disabled="pagination.page === 1">
          &lt;
        </a>
      </li>
      <li class="page-item" :class="{ active: pagination.page === index + 1 }" v-for="(page, index) in totalPages"
        :key="index">
        <a class="page-link" @click="changePage(index + 1)" href="#">{{ index + 1 }}</a>
      </li>
      <li class="page-item">
        <a class="page-link" @click="changePage(pagination.page + 1)" href="#" :disabled="pagination.page === totalPages">
          &gt;
        </a>
      </li>
    </ul>
  </nav>
  <footer>
    <p>&copy; 2024 MANNIT-INNOVATIONS</p>
  </footer>
</template>

<script>
import Header from '~/components/Header.vue';
import axios from 'axios';

export default {
  components: {
    Header,
  },
  data() {
    return {
      searchDate: new Date().toISOString().split('T')[0],  // Initialize with the current date
      reportData: [],  // Initialize as an empty array
      pagination: {
        page: 1,
        itemsPerPage: 5,
      },
      originalReportData: [],
      selectedTable: 'patientBooking',
    };
  },
  created() {
    this.searchReport();  // Call searchReport on component creation
  },
  computed: {
    paginatedItems() {
      // Ensure that reportData is an array before calling slice
      const dataToPaginate = Array.isArray(this.reportData) ? this.reportData : [];
      const start = (this.pagination.page - 1) * this.pagination.itemsPerPage;
      const end = start + this.pagination.itemsPerPage;
      return dataToPaginate.slice(start, end);
    },
    totalPages() {
      // Ensure that reportData is an array before calculating totalPages
      const dataToPaginate = Array.isArray(this.reportData) ? this.reportData : [];
      return Math.ceil(dataToPaginate.length / this.pagination.itemsPerPage);
    },
  },
  methods: {
    async searchReport() {
      // Validate the date
      if (!this.searchDate) {
        console.error('Please choose a date.');
        return;
      }

      try {
        console.log('Making API request with date:', this.searchDate);
        const response = await axios.get(`http://localhost:8081/api/getbydate?date=${this.searchDate}`);
        this.originalReportData = response.data;
        this.pagination.page = 1;

        // Toggle table based on selectedTable
        switch (this.selectedTable) {
          case 'patientBooking':
            this.reportData = this.originalReportData.currentPatients;
            break;
          case 'queriedPatient':
            this.reportData = this.originalReportData.queriedPatients;
            break;
          case 'rejectedPatient':
            this.reportData = this.originalReportData.noAppointments;
            break;
        }
        this.reportData.sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));
      } catch (error) {
        console.error('Error fetching data:', error.message);
      }
    },
    updateDate(event) {
      this.searchDate = event.target.value;
    },
    toggleTable(tableName) {
      this.selectedTable = tableName;
    },
    changePage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.pagination.page = page;
      }
    },
  },
};
</script>


<style scoped>
@import '@/styles/chatbot.css';
</style>
