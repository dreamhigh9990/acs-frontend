<template>
  <div class="d-flex flex-column flex-grow-1 pt-8">
    <create-report
      v-if="!creatingReport"
      @createReport="creatingReport = true"
    >
    </create-report>
    <v-stepper v-else v-model="stepNumber" alt-labels>
      <v-stepper-header>
        <v-stepper-step :complete="stepNumber > 1" step="1">Select Location</v-stepper-step>

        <v-divider></v-divider>

        <v-stepper-step :complete="stepNumber > 2" step="2">Select Zone</v-stepper-step>

        <v-divider></v-divider>

        <v-stepper-step :complete="stepNumber > 3" step="3">Select Machines</v-stepper-step>

        <v-divider></v-divider>

        <v-stepper-step :complete="stepNumber > 4" step="4">Select Parameters</v-stepper-step>

        <v-divider></v-divider>

        <v-stepper-step :complete="stepNumber > 5" step="5">Select Date</v-stepper-step>

        <v-divider></v-divider>

        <v-stepper-step :complete="stepNumber > 6" step="6">Generate Report</v-stepper-step>
      </v-stepper-header>

      <v-stepper-items>
        <v-stepper-content step="1">
          <select-location
            @setLocationId="handleSetLocation"
          >
          </select-location>
        </v-stepper-content>

        <v-stepper-content step="2">
          <select-zone
            @setZoneId="handleSetZone"
            @cancel="handeCancel"
          >
          </select-zone>
        </v-stepper-content>

        <v-stepper-content step="3">
          <select-machines
            ref="selectMachines"
            @setReportMachines="handleSetReportMachines"
            @cancel="handeCancel"
          >
          </select-machines>
        </v-stepper-content>

        <v-stepper-content step="4">
          <select-tags
            :device-ids="deviceIds"
            :selected-tags="selectedTags"
            @setMachineTags="handleSetMachineTags"
            @cancel="handeCancel"
          >
          </select-tags>
        </v-stepper-content>

        <v-stepper-content step="5">
          <select-time-range
            :selected-tags="selectedTags"
            @setSelectedTimeRange="handleSetTimeRange"
            @cancel="handeCancel"
          >
          </select-time-range>
        </v-stepper-content>

        <v-stepper-content step="6">
          <generate-report
            :selected-tags="selectedTags"
            :selected-time-range="selectedTimeRange"
            :report-title="reportTitle"
            @generateReport="handleGenerateReport"
            @cancel="handeCancel"
            @handleResetCreating="handleReset"
          >
          </generate-report>
        </v-stepper-content>
      </v-stepper-items>
    </v-stepper>
    <reports-list-page></reports-list-page>
  </div>
</template>
<script>
import { mapState, mapActions } from 'vuex'

import SelectLocation from './components/SelectLocation'
import SelectZone from './components/SelectZone'
import CreateReport from './components/CreateReport'
import SelectMachines from './components/SelectMachines'
import SelectTags from './components/SelectTags'
import SelectTimeRange from './components/SelectTimeRange'
import GenerateReport from './components/GenerateReport'
import ReportsListPage from './ReportsListPage'

export default {
  components: {
    SelectLocation,
    SelectZone,
    CreateReport,
    SelectMachines,
    SelectTags,
    SelectTimeRange,
    GenerateReport,
    ReportsListPage
  },
  data() {
    return {
      creatingReport: false,
      deviceIds: [],
      locationId: 0,
      zoneId: 0,
      selectedTags: {},
      selectedTimeRange: {},
      timeRange: {},
      reportTitle: '',
      stepNumber: 1,
      errorMessage: ''
    }
  },
  methods: {
    ...mapActions({
      getMachineTags: 'machines/getMachineTags',
      generateMachinesReport: 'machines/generateMachinesReport',
      initZonesTable: 'machines/initZonesTable',
      getMachines: 'machines/getMachines',
      getReportsList: 'machines/getReportsList',
      showErrorMessage(dispatch, message) {
        return dispatch('app/showError', message, { root: true })
      }
    }),
    handleSetLocation(locationId) {
      this.locationId = locationId
      this.stepNumber = 2
      try {
        this.initZonesTable(this.locationId)
      } catch (error) {
        console.log(error)
      }
    },
    async handleSetZone(zoneId) {
      this.zoneId = zoneId
      this.stepNumber = 3
      try {
        await this.getMachines({
          location: this.locationId,
          zone: this.zoneId
        })
      } catch (error) {
        console.log(error)
      }
      this.$refs.selectMachines.resetModel()
    },
    handleSetReportMachines(data) {
      this.selectedTags = {}
      this.deviceIds = data
      this.stepNumber = 4
      try {
        this.getMachineTags({
          deviceIds: data
        })
      } catch (error) {
        console.log(error)
      }
    },
    handleSetMachineTags(data) {
      this.selectedTags = data
      this.stepNumber = 5
    },
    handleSetTimeRange(data) {
      this.selectedTimeRange = data
      this.stepNumber = 6
    },
    async handleGenerateReport(data, title) {
      this.timeRange = data
      try {
        this.errorMessage = ''
        await this.generateMachinesReport({
          machineTags: this.selectedTags,
          timeRange: this.timeRange,
          reportTitle: title
        })

        this.getReportsList()
      } catch (error) {

        this.errorMessage = 'Failed to export report due to selected data size. Please choose a smaller timeframe and a smaller selection of devices and data points.'

        this.$store.dispatch('app/showError', { message: 'Error: ', error: { message: this.errorMessage } }, { root: true })
        console.log(error)
      }

    },
    handeCancel() {
      this.errorMessage = ''
      this.stepNumber -= 1
    },
    handleReset() {
      this.errorMessage = ''
      this.creatingReport = false
      this.stepNumber = 1
    }
  }
}
</script>
