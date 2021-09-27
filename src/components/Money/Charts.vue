<template>
  <div class="wrapper" ref="wrapper">Charts</div>
</template>
<script lang="ts">
import Vue from 'vue';
import {Component, Prop, Watch} from 'vue-property-decorator';
import * as echarts from 'echarts';
import {EChartsOption} from 'echarts';

@Component
export default class Charts extends Vue {
  @Prop() options?: EChartsOption;
  chart: ECharts;

  mounted() {
    this.chart = echarts.init(this.$refs.wrapper);
    this.chart.setOption(this.options)
  }

  @Watch('options')
  onOptionsChanged(newValue) {
    this.chart.setOption(newValue);
  }
}
</script>

<style lang="scss" scoped>
.wrapper {
  height: 400px;
}
</style>