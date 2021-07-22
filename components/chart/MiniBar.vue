<template>
  <div class="mini-chart">
    <div class="chart-content" :style="{ height: 88 }">
      <v-chart
        :force-fit="true"
        :height="height"
        :data="data"
        :padding="[36, 5, 18, 5]"
      >
        <v-tooltip />
        <v-legend />
        <v-bar position="日期*數量" color="type" :adjust="adjust" />
      </v-chart>
    </div>
  </div>
</template>

<script>
//import { format } from 'date-fns'
import DataSet from '@antv/data-set'

const sourceData = [
  {
    type: 'input',
    '1': 10,
    '2': 12,
    '3': 39,
    '4': 81,
    '5': 47,
    '6': 20,
    '7': 24,
    '8': 35,
    '9': 24,
    '10': 35,
    '11': 10,
    '12': 12,
    '13': 39,
    '14': 81,
    '15': 47,
    '16': 20,
    '17': 24,
    '18': 35,
    '19': 24,
    '20': 35,
    '21': 10,
    '22': 12,
    '23': 39,
    '24': 81,
    '25': 47,
    '26': 20,
    '27': 24,
    '28': 35,
    '29': 24,
    '30': 35,
  },
  {
    type: 'output',
    '1': 12,
    '2': 23,
    '3': 34,
    '4': 99,
    '5': 52,
    '6': 35,
    '7': 37,
    '8': 42,
    '9': 47,
    '10': 52,
    '11': 12,
    '12': 23,
    '13': 34,
    '14': 99,
    '15': 52,
    '16': 35,
    '17': 37,
    '18': 42,
    '19': 47,
    '20': 52,
    '21': 12,
    '22': 23,
    '23': 34,
    '24': 99,
    '25': 52,
    '26': 35,
    '27': 37,
    '28': 42,
    '29': 47,
    '30': 52,
  },
]
const dv = new DataSet.View().source(sourceData)
dv.transform({
  type: 'fold',
  fields: [
    '1',
    '2',
    '3',
    '4',
    '5',
    '6',
    '7',
    '8',
    '9',
    '10',
    '11',
    '12',
    '13',
    '14',
    '15',
    '16',
    '17',
    '18',
    '19',
    '20',
    '21',
    '22',
    '23',
    '24',
    '25',
    '26',
    '27',
    '28',
    '29',
    '30',
  ],
  key: '日期',
  value: '數量',
})
const data = dv.rows
const tooltip = [
  'x*y',
  (x, y) => ({
    name: x,
    value: y,
  }),
]

const scale = [
  {
    dataKey: 'x',
    min: 2,
  },
  {
    dataKey: 'y',
    title: '时间',
    min: 1,
    max: 22,
  },
  {
    dataKey: 'z',
    title: '时间S',
    min: 1,
    max: 22,
  },
]

export default {
  name: 'MiniBar',
  data() {
    return {
      data,
      scale,
      tooltip,
      height: 140,
      adjust: [
        {
          type: 'dodge',
          marginRatio: 1 / 32,
        },
      ],
    }
  },
}
</script>

<style lang="less" scoped>
@import 'index.less';
</style>
