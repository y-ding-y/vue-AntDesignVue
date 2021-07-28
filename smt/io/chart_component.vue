<template>
  <div>
    <ef-chart class="basebg" :padding="[24, 24, 48, 64]" :scale="scale" :series="series" :data="chart_data"
      :height="height">
    </ef-chart>
  </div>
</template>

<script>
  import EfChart from '@/components/chart/Chart'
  import { merge } from '@/utils/util'
  const DataSet = require('@antv/data-set')

  export default {
    name: "g5io_chart",
    i18n: merge(require('@/i18n'), require('./i18n')),
    props: {
      actionFix: String,
      groupkey: String,
      groupext: Object,
      data: Array,
      chartHeight: String,
      rcnames: Object,
    },
    data() {
      return {}
    },
    components: {
      EfChart,
    },
    computed: {
      chart_data() {
        var rcs = [];
        for (var key in this.rcnames) {
          rcs.push({ rc: key, rcname: this.rcnames[key] })
        }

        const data = this.data
          .map((e) => {
            e.qty = parseInt(e.qty)
            e.rc = String(e.rc)
            return {
              rc: e.rc,
              rcname: e.rcname,
              qty: e.qty,
              name: e.name,
              key: String(e.key),
            }
          })
          .sort((a, b) => {
            if (a.key != b.key) {
              return a.key > b.key ? 1 : -1
            }
            const aI = rcs.findIndex((e) => e.rc + '' === a.rc)
            const bI = rcs.findIndex((e) => e.rc + '' === b.rc)

            return Math.sign(aI - bI)
          })
        const ds = new DataSet()
        const dv = ds
          .createView()
          .source(data)
          .transform({
            type: 'sort-by',
            fields: ['key'], // 根据指定的字段集进行排序，与lodash的sortBy行为一致
            order: 'ASC', // 默认为 ASC，DESC 则为逆序
          })
        return dv.rows
      },

      scale() {
        return [
          {
            dataKey: 'qty',
          },
          {
            dataKey: 'key',
            type: 'cat',
          },
        ]
      },

      series() {
        return [
          {
            quickType: 'bar',
            position: this.groupext[this.groupkey] ? 'name*qty' : 'key*qty',
            color: 'rcname',
            adjust: [
              {
                type: 'dodge',
                marginRatio: 1 / 32,
              },
            ],
          },
        ]
      },

      height() {
        return this.chartHeight === 'default'
          ? 400
          : this.chartHeight === 'middle'
            ? 300
            : 200
      },
    },

    methods: {},
  }
</script>

<style scoped lang="less">
  .basebg {
    background-color: @base-bg-color;
  }
</style>