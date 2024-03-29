<template>
  <div class="d-flex flex-grow-1 flex-column">
    <v-sheet color="surface2" class="my-n8 pt-9 py-7">
      <v-container class="pb-0">
        <div v-if="$route.name === 'acs-machines'" class="d-flex mt-2 align-center">
          <v-breadcrumbs :items="acsBreadcrumbItems">
            <template v-slot:item="{ item }">
              <v-breadcrumbs-item
                :disabled="item.disabled"
              >
                {{ item.text }}
              </v-breadcrumbs-item>
            </template>
          </v-breadcrumbs>
          <v-spacer></v-spacer>
          <company-menu
            :companies="companies"
            @companyChanged="onCompanyChanged"
          >
          </company-menu>
        </div>
        <v-breadcrumbs v-else :items="breadcrumbItems"></v-breadcrumbs>
        <top-card></top-card>
      </v-container>
    </v-sheet>

    <v-container>
      <dashboard-table
        :loading="loadingLocationsTable"
        :items="locations"
        table-type="location"
      >
      </dashboard-table>

      <br>
      <saved-machines-table-card></saved-machines-table-card>
      <br>
      <machines-table-card></machines-table-card>
    </v-container>
  </div>
</template>

<script>

/*
|---------------------------------------------------------------------
| Dashboard Page Component
| url: /dashboard/analytics
|---------------------------------------------------------------------
|
*/

import { mapState, mapGetters, mapActions } from 'vuex'

import CompanyMenu from '../../components/dashboard/CompanyMenu'
import TopCard from '../../components/dashboard/TopCard'
import MachinesTableCard from '../../components/dashboard/MachinesTableCard'
import DashboardTable from '../../components/dashboard/dashboard-tables/DashboardTable'
import SavedMachinesTableCard from '../../components/dashboard/SavedMachinesTableCard'

export default {
  components: {
    CompanyMenu,
    MachinesTableCard,
    DashboardTable,
    TopCard,
    SavedMachinesTableCard
  },
  data() {
    return {
    }
  },
  computed: {
    ...mapState({
      loadingLocationsTable: (state) => state.machines.loadingLocationsTable,
      loadingDashboardDevicesTable: (state) => state.devices.loadingDashboardDevicesTable,

      locations: (state) => state.locations.data,
      companies: (state) => state.companies.companies,
      selectedCompanyName: (state) => state.machines.selectedCompany ? state.machines.selectedCompany.name : '',
      selectedCompany: (state) => state.machines.selectedCompany,
      userCompanyName: (state) => state.auth.user.companyName
    }),
    ...mapGetters('auth', ['canViewCompanies', 'isAcsUser']),
    acsBreadcrumbItems() {
      return [
        {
          text: this.selectedCompanyName,
          disabled: true
        }
      ]
    },
    breadcrumbItems() {
      return [
        {
          text: this.userCompanyName,
          disabled: true
        }
      ]
    }
  },
  async mounted() {
    if (this.canViewCompanies)
      await this.initAcsDashboard()
    this.getZones()
    this.initLocationsTable({ companyId: this.selectedCompany ? this.selectedCompany.id : 0 })

    if (!(this.isAcsUser && this.selectedCompany && this.selectedCompany.id === 0)) {
      this.getAlarmsReports({
        companyId: this.selectedCompany ? this.selectedCompany.id : 0
      })
    }

    this.getDowntimeGraphData({
      company_id: this.selectedCompany ? this.selectedCompany.id : 0,
      location_id: 0,
      to: new Date().getTime(),
      from: new Date().getTime() - 60 * 60 * 24 * 1000
    })

    this.getDowntimeByTypeGraphSeries({
      company_id: this.selectedCompany ? this.selectedCompany.id : 0,
      location_id: 0,
      to: new Date().getTime(),
      from: new Date().getTime() - 60 * 60 * 24 * 1000
    })

    this.getDowntimeByReasonGraphSeries({
      company_id: this.selectedCompany ? this.selectedCompany.id : 0,
      location_id: 0,
      to: new Date().getTime(),
      from: new Date().getTime() - 60 * 60 * 24 * 1000
    })
  },
  methods: {
    ...mapActions({
      initAcsDashboard: 'machines/initAcsDashboard',
      initLocationsTable: 'machines/initLocationsTable',
      getZones: 'zones/getZones',
      changeSelectedCompany: 'machines/changeSelectedCompany',
      getDevicesAnalytics: 'devices/getDevicesAnalytics',
      getAlarmsReports: 'alarms/getAlarmsReports',
      getDowntimeGraphData: 'devices/getDowntimeGraphData',
      getDowntimeByTypeGraphSeries: 'devices/getDowntimeByTypeGraphSeries',
      getDowntimeByReasonGraphSeries: 'devices/getDowntimeByReasonGraphSeries'
    }),
    onCompanyChanged(company) {
      this.changeSelectedCompany(company)

      this.initLocationsTable({
        companyId: this.selectedCompany ? this.selectedCompany.id : 0
      })

      this.getDevicesAnalytics({
        page: 1,
        location_id: this.location,
        company_id: this.selectedCompany ? this.selectedCompany.id : 0
      })

      if (!(this.isAcsUser && this.selectedCompany && this.selectedCompany.id === 0)) {
        this.getAlarmsReports({
          companyId: this.selectedCompany ? this.selectedCompany.id : 0
        })
      }

      this.getDowntimeGraphData({
        company_id: this.selectedCompany ? this.selectedCompany.id : 0,
        location_id: 0,
        zone_id: 0,
        to: new Date().getTime(),
        from: new Date().getTime() - 60 * 60 * 24 * 1000
      })

      this.getDowntimeByTypeGraphSeries({
        company_id: this.selectedCompany ? this.selectedCompany.id : 0,
        location_id: 0,
        zone_id: 0,
        to: new Date().getTime(),
        from: new Date().getTime() - 60 * 60 * 24 * 1000
      })

      this.getDowntimeByReasonGraphSeries({
        company_id: this.selectedCompany ? this.selectedCompany.id : 0,
        location_id: 0,
        zone_id: 0,
        to: new Date().getTime(),
        from: new Date().getTime() - 60 * 60 * 24 * 1000
      })
    }
  }
}
</script>
