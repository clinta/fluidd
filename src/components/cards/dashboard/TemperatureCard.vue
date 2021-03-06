<template>
  <collapsable-card
    title="Thermals"
    icon="$fire"
    :lazy="false"
    :draggable="true"
    :inLayout="inLayout"
    :enabled="enabled"
    @enabled="$emit('enabled', $event)">

    <temperature-targets-widget
      @legendClick="setLegend"
      @legendPowerClick="setLegendPower">
    </temperature-targets-widget>

    <v-card-text class="py-0" :class="{ 'mb-4': !chartVisible }">
      <v-btn
        dark
        small
        text
        outlined
        block
        color="secondary"
        @click="chartVisible = !chartVisible"
      >
        <v-icon dark v-if="chartVisible">$minus</v-icon>
        <v-icon dark v-else>$plus</v-icon>
        {{ (chartVisible) ? 'hide graph' : 'show graph' }}
      </v-btn>
    </v-card-text>

    <e-charts-widget v-if="chartReady && chartVisible" :chart-data="chartData" ref="chart"></e-charts-widget>

  </collapsable-card>
</template>

<script lang="ts">
import { Component, Mixins, Prop } from 'vue-property-decorator'
import TemperatureTargetsWidget from '@/components/widgets/TemperatureTargetsWidget.vue'
import EChartsWidget from '@/components/widgets/EChartsWidget.vue'
import UtilsMixin from '@/mixins/utils'
import { Fan, Heater } from '@/store/socket/types'

@Component({
  components: {
    TemperatureTargetsWidget,
    EChartsWidget
  }
})
export default class TemperatureGraphCard extends Mixins(UtilsMixin) {
  @Prop({ type: Boolean, default: true })
  enabled!: boolean

  get chartData () {
    return this.$store.getters['socket/getChartData']
  }

  get chartReady () {
    return (
      this.$store.state.socket.acceptingNotifications &&
      this.$store.state.socket.ready &&
      this.klippyConnected
    )
  }

  setLegend (item: Heater | Fan) {
    const ref = this.$refs.chart as EChartsWidget
    ref.legendToggle(item.name)
  }

  setLegendPower (item: Heater | Fan) {
    const ref = this.$refs.chart as EChartsWidget
    const name = ('speed' in item)
      ? item.name + 'Speed'
      : item.name + 'Power'
    const legend = ref.selectedLegends[name]
    if (legend) {
      ref.legendUnSelect(name)
    } else {
      ref.legendSelect(name)
    }
  }

  get chartVisible () {
    return this.$store.state.config.fileConfig.general.chartVisible
  }

  set chartVisible (value: boolean) {
    this.$store.dispatch('config/saveGeneric', { key: 'fileConfig.general.chartVisible', value })
  }

  get inLayout (): boolean {
    return (this.$store.state.config.layoutMode)
  }
}
</script>
