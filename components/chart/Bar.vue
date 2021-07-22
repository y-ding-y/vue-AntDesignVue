<template>
  <div class="bar">
    <h4 v-if="title != ''" :style="titleStyle">{{ title }}</h4>
    <div class="chart">
      <v-chart
        :forceFit="forceFit"
        :height="height"
        :data="data"
        :padding="padding"
      >
        <v-tooltip />
        <v-axis />
        <v-legend />

        <v-series quickType="bar" position="x*y" />
        <!--v-line position="x*z" />
        <v-point position="x*z" /-->
      </v-chart>
    </div>
  </div>
</template>

<script>
const data = []
for (let i = 8; i < 17; i += 1) {
  data.push({
    x: `${i + 1}:00`,
    y: Math.floor(Math.random() * 10) + 50,
    z: (Math.floor(Math.random() * 5) + 80) / 100,
  })
}
const tooltip = [
  'x*y',
  (x, y) => ({
    name: x,
    value: y,
  }),
  'x*z',
  (x, z) => ({
    name: x,
    value: z,
  }),
]

export default {
  name: 'Bar',
  props: {
    title: String,
    titleStyle: { type: [String, Object], default: '' },
    forceFit: { type: Boolean, default: true },
    height: { type: Number, default: 314 * 0 + 240 },
    padding: { type: Array, default: () => [24, 42] },
    scale: {
      type: Array,
      default: () => [
        {
          dataKey: 'x',
          min: 1,
        },
        {
          dataKey: 'y',
          title: '时间',
          min: 1,
          max: 22,
        },
      ],
    },
    tooltip: {
      type: Array,
      default: () => tooltip,
    },
    data: {
      type: [Object, Array],
      default: () => data,
    },
  },
  data() {
    return {}
  },
}
</script>

<style scoped lang="less">
.bar {
  position: relative;
  .chart {
  }
}
</style>
