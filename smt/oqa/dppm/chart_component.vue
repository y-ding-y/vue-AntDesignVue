<template>
    <div>
        <ef-chart class="basebg" :padding="[24, 54, 48, 64]" :scale="scale" :series="series" :data="chart_data"
            :height="height">
        </ef-chart>

    </div>
</template>

<script>

    import EfChart from '@/components/chart/Chart'
    import { merge } from '@/utils/util'
    const DataSet = require('@antv/data-set')
    const arr = [
        'class',
        'pn',
        'wo',
        'month',
        'week',
        'date',
        'hour',];
    export default {
        name: "dppm_chart",
        i18n: merge(require('@/i18n'), require('./i18n')),
        props: {
            actionFix: String,
            groupkey: String,
            groupext: Object,
            data: Array,
            chartHeight: String,
        },
        data() {
            return {
                arr
            }
        },
        components: {
            EfChart
        },
        computed: {
            chart_data() {
                const data = this.data.slice(0, -1).map((e) => {
                    // e.qty = parseInt(e.qty)
                    // e.rc = String(e.route_code)
                    return {
                        dppm: e.dppm,
                        name: e.name,
                        rate: e.rate,
                        [this.$t('sample')]: e.sample,
                        [this.$t('fail')]: e.fail,
                        [this.$t('miss')]: e.miss,
                        key: String(e.key)
                    }

                })
                const ds = new DataSet();
                const dv = ds
                    .createView()
                    .source(data)

                    .transform({
                        // type: 'sort-by',
                        // fields: ["key"], // 根据指定的字段集进行排序，与lodash的sortBy行为一致
                        // order: 'ASC', // 默认为 ASC，DESC 则为逆序
                        type: 'fold',
                        fields: [this.$t('sample'), this.$t('fail'), this.$t('miss')],
                        key: 'type',
                        value: 'qty',
                    })
                return dv.rows;
            },

            scale() {
                return [
                    { dataKey: "dppm", alias: this.$t('dppm'), },
                    { dataKey: 'key', type: 'cat' },
                ]
            },

            series() {
                return [
                    {
                        quickType: 'bar',
                        position: this.groupext[this.groupkey] ? 'name*qty' : 'key*qty',
                        color: 'type',
                        adjust: [
                            {
                                type: 'dodge',
                                marginRatio: 1 / 32,
                            },
                        ],
                    },
                    {
                        quickType: 'line',
                        position: this.groupext[this.groupkey] ? "name*dppm" : 'key*dppm', color: "#4FAAEB",
                    },
                    {
                        quickType: 'point',
                        position: this.groupext[this.groupkey] ? "name*dppm" : 'key*dppm', color: "#4FAAEB",
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

        methods: {

        }
    }
</script>

<style scoped lang="less">
    .basebg {
        background-color: @base-bg-color;
    }
</style>