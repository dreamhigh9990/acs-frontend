<template>
  <v-card
    max-width="900"
    class="mx-auto"
  >
    <v-card-title class="primary white--text">Alarm Reports</v-card-title>
    <v-list
      subheader
      two-line
    >
      <template v-for="(alarm, index) in activeAlarms">
        <v-list-item
          :key="`${alarm.alarm_name}${alarm.device_id}`"
        >
  
          <v-list-item-content>
            <v-list-item-title v-text="alarm.machine_name"></v-list-item-title>
  
            <v-list-item-subtitle class="text--primary" v-text="alarm.device_name"></v-list-item-subtitle>
            <v-list-item-subtitle class="red--text" v-text="alarm.alarm_name"></v-list-item-subtitle>
          </v-list-item-content>
  
          <v-list-item-action>
            <v-tooltip bottom>
              <template v-slot:activator="{ on, attrs }">
                <v-btn icon @click="handleClick(alarm)">
                  <v-icon
                    color="primary lighten-1"
                    v-bind="attrs"
                    v-on="on"
                  >$mdi-eye</v-icon>
                </v-btn>
              </template>
              <span>See Machine Details</span>
            </v-tooltip>
          </v-list-item-action>
          
        </v-list-item>
        <v-divider
          v-if="index < activeAlarms.length - 1"
          :key="index"
        ></v-divider>
      </template>
    </v-list>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn color="primary" @click="$emit('close')">Close</v-btn>
    </v-card-actions>
  </v-card>
</template>
<script>
export default {
  props: {
    activeAlarms: {
      type: Array,
      default: () => {
        return []
      }
    }
  },
  data() {
    return {
    }
  },
  methods: {
    handleClick(item) {
      if (item.location_id && item.zone_id) {
        this.$router.push({
          name: 'dashboard-product',
          params: {
            location: item.location_id,
            zone: item.zone_id,
            configurationId: item.machine_id,
            productId: item.device_id
          }
        })
      } else {
        this.$router.push({
          name: 'product-details',
          params: {
            configurationId: item.machine_id,
            productId: item.device_id
          }
        })
      }

      this.$emit('close')
    }
  }
}
</script>
