<template>
  <v-card
    class="d-flex flex-column justify-space-between"
    height="100%"
    light
    :loading="isDowntimeByTypeGraphLoading"
    :disabled="isDowntimeByTypeGraphLoading"
  >
    <v-card-title>
      Downtime by Type
      <v-spacer></v-spacer>
      <v-btn
        class="ml-1"
        color="primary"
        @click="showDowntimeTable=!showDowntimeTable"
      >
        Downtime Administration
      </v-btn>
      <v-dialog
        v-model="showDowntimeTable"
        max-width="1200px"
      >
        <downtime-data-table @closed="showDowntimeTable=false"></downtime-data-table>
      </v-dialog>
    </v-card-title>
    <v-card-text>
      <apexchart
        :options="chartOptions"
        height="400"
        :series="getDowntimeByTypeSeries"
      ></apexchart>
    </v-card-text>
  </v-card>
</template>

<script>
import { mapActions, mapState, mapGetters } from 'vuex'
import DowntimeDataTable from './product/DowntimeDataTable'
export default {
  components: {
    DowntimeDataTable
  },
  data() {
    return {
      selectedTimeRange: {},
      showChart: true,
      showDowntimeTable: false
    }
  },
  computed: {
    ...mapState({
      downtimeByTypeGraphSeries: (state) => state.devices.downtimeByTypeGraphSeries,
      isDowntimeByTypeGraphLoading: (state) => state.devices.isDowntimeByTypeGraphLoading
    }),
    ...mapGetters('machines', ['timeRangeFromTo']),
    chartOptions() {
      return {
        chart: {
          type: 'bar',
          horizontal: false,
          stacked: true,
          toolbar: {
            show: false
          },
          zoom: {
            enabled: true
          }
        },
        plotOptions: {
          bar: {
            horizontal: false,
            columnWidth: '60%',
            dataLabels: {
              show: false,
              position: 'top'
            },
            distributed: true,
            space: 0.25,
            endingShape: 'rounded'
          }
        },
        xaxis: {
          categories: this.downtimeByTypeGraphSeries.map((item) => item.name),
          labels: {
            show: false
          }
        },
        legend: {
          position: 'bottom',
          offsetY: 10
        },
        fill: {
          opacity: 1
        }
      }
    },
    getDowntimeByTypeSeries() {
      const series = [{ name: 'Hours', data: [] }]

      this.downtimeByTypeGraphSeries.map((item) => {
        series[0].data.push(item.data)

        return 0
      })

      return series
    }
  }
}
</script>
