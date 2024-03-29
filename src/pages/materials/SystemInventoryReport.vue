<template>
  <div class="d-flex flex-column flex-grow-1">
    <v-stepper v-model="stepNumber" alt-labels :disabled="loadingReports" :loading="loadingReports">
      <v-stepper-header>
        <v-stepper-step :complete="stepNumber > 1" step="1">Select Location</v-stepper-step>

        <v-divider></v-divider>

        <v-stepper-step :complete="stepNumber > 2" step="2">Select Date</v-stepper-step>

        <v-divider></v-divider>

        <v-stepper-step :complete="stepNumber > 3" step="3">Generate Report</v-stepper-step>
      </v-stepper-header>

      <v-stepper-items>
        <v-stepper-content step="1">
          <select-location
            @setLocationId="handleSetLocation"
          >
          </select-location>
        </v-stepper-content>

        <v-stepper-content step="2">
          <select-time-range
            :location-id="locationId"
            @setSelectedTimeRange="handleSetTimeRange"
            @cancel="handeCancel"
          >
          </select-time-range>
        </v-stepper-content>

        <v-stepper-content step="3">
          <generate-report
            :location-id="locationId"
            :selected-time-range="selectedTimeRange"
            @generateReport="handleGenerateReport"
            @search="handleSearch"
            @cancel="handeCancel"
            @handleResetCreating="handleReset"
          >
          </generate-report>
        </v-stepper-content>
      </v-stepper-items>
    </v-stepper>
    <v-card v-if="stepNumber > 2" class="mt-2" :disabled="loadingReports" :loading="loadingReports">
      <v-card-title>Material Usage Report</v-card-title>

      <v-card-text>
        <v-data-table
          :headers="tableHeaders"
          :items="systemInventoryReports"
          class="flex-grow-1"
        >
          <template v-slot:item.value="{ item }">
            {{ item.value.toFixed(3) }}
          </template>
        </v-data-table>
      </v-card-text>
    </v-card>
  </div>
</template>
<script>
import { mapState, mapGetters, mapActions } from 'vuex'

import SelectLocation from './components/SelectLocation'
import SelectTimeRange from './components/SelectTimeRange'
import GenerateReport from './components/GenerateReport'

export default {
  components: {
    SelectLocation,
    SelectTimeRange,
    GenerateReport
  },
  data() {
    return {
      locationId: 0,
      selectedTimeRange: {},
      timeRange: {},
      stepNumber: 1,
      errorMessage: '',
      tableHeaders: [
        { text: 'Material', value: 'material' },
        { text: 'Material Used(lbs)', value: 'value' }
      ]
    }
  },
  computed: {
    ...mapState({
      loadingReports: (state) => state.materials.loadingReports,
      exportingReport: (state) => state.materials.exportingReport,

      systemInventoryReports: (state) => state.materials.systemInventoryReports
    }),
    ...mapGetters('machines', ['timeRangeFromTo'])
  },
  methods: {
    ...mapActions({
      getSystemInventoryReport: 'materials/getSystemInventoryReport',
      exportSystemInventoryReport: 'materials/exportSystemInventoryReport',
      generateMachinesReport: 'machines/generateMachinesReport',
      initZonesTable: 'machines/initZonesTable',
      getReportsList: 'machines/getReportsList',
      showErrorMessage(dispatch, message) {
        return dispatch('app/showError', message, { root: true })
      }
    }),
    handleSetLocation(locationId) {
      this.locationId = locationId
      this.stepNumber = 2
    },
    handleSetTimeRange(data) {
      this.selectedTimeRange = data
      this.handleSearch(this.getTimeRange())
      this.stepNumber = 3
    },
    async handleGenerateReport(data) {
      this.timeRange = data
      this.errorMessage = ''

      try {
        const response = await this.exportSystemInventoryReport({
          location: this.locationId,
          timeRange: this.timeRange
        })

        const filepath = this.$REPORTS_URL + response.filename

        this.$download(filepath, response.filename)
      } catch (err) {
        console.log(err)
      }
    },
    async handleSearch(data) {
      this.timeRange = data
      try {
        this.errorMessage = ''
        this.getSystemInventoryReport({
          location: this.locationId,
          timeRange: this.timeRange
        })
      } catch (error) {
        console.log('error:', error)
      }
    },
    handeCancel() {
      this.errorMessage = ''
      this.stepNumber -= 1
    },
    handleReset() {
      this.errorMessage = ''
      this.stepNumber = 1
    },
    getTimeRange() {
      if (this.selectedTimeRange && this.selectedTimeRange.timeRangeOption !== 'custom') {
        const tR = {
          timeRangeOption: this.selectedTimeRange.timeRangeOption,
          dates: [new Date().toISOString().substr(0, 10), new Date().toISOString().substr(0, 10)]
        }

        const from = new Date(this.timeRangeFromTo(tR).from).toISOString()
        const to =  new Date(this.timeRangeFromTo(tR).to).toISOString()

        const timeRange = {
          dateFrom: from.substr(0, 10),
          dateTo: to.substr(0, 10),
          timeFrom: from.substr(11, 8),
          timeTo: to.substr(11, 8)
        }

        return timeRange
      } else {
        const timeRange = {
          dateFrom: this.selectedTimeRange.dateFrom,
          dateTo: this.selectedTimeRange.dateTo,
          timeFrom: this.selectedTimeRange.timeFrom,
          timeTo: this.selectedTimeRange.timeTo
        }

        return timeRange
      }
    }
  }
}
</script>
