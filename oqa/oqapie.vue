<template>
    <div>
        <a-card>
            <h4 v-if="title" v-bind:style="titleStyle">{{ title }}
            </h4>
            <a-spin :spinning="loading"></a-spin>
            <v-chart :forceFit="true" :data="chartData" :height="height" :scale="[
        {
          dataKey: 'percent',
          formatter: '.0%',
        },
      ]" :padding="[0,0,0,0]">
                <v-tooltip :showTitle="false" dataKey="name*qty" />
                <v-axis />
                <v-legend dataKey="name" v-if="false" />
                <v-pie position="qty" color="name" :vStyle="pieStyle" :label="labelConfig" />
                <v-coord type="theta" :radius="0.55" :innerRadius="0.4" />
            </v-chart>
        </a-card>
    </div>
</template>

<script>
    import { reqPost } from '@/services/base'
    //import EfMain from '@/components/page/main'
    // import Chart from '@/components/chart/Chart'
    import { merge } from '@/utils/util'
    const DataSet = require('@antv/data-set')



    export default {
        name: 'oqachart',
        i18n: merge(require('@/i18n'), require('./i18n')),
        //components: { EfChart: Chart, },
        props: {
            type: { type: String, default: 'caustomer' },
            category: { type: [String,Array], default: '114' },
            title: String,
            paras: Object
        },
        data() {
            return {
                loadcount: 0,
                sSize: 'middle',
                chartData: [],
                pieStyle: {
                    stroke: '#fff',
                    lineWidth: 0.5,
                },
                height: 200,
                titleStyle: { textAlign: "center", color: "black", fontSize: "20px" },
            }
        },
        computed: {
            // title() {
            //     return this.$t('defectTitle')
            // },
            loading() {
                return this.loadcount > 0
            },
            chartHeight() {
                return this.sSize === 'default'
                    ? 400
                    : this.sSize === 'middle'
                        ? 300
                        : 200
            },
            labelConfig() {
                return [
                    'percent',
                    {
                        formatter: (val, item) => {
                            return item.point.name + ':' + val
                        },
                        label: {
                            textStyle: {
                                fontSize: 14,
                                fontWeight: 400
                            },
                        },

                    },
                ]
            },
        },
        watch: {},
        created() {
            this.loadcount = 1;
            reqPost("FILE_ACTION", { prop: "pie", type: this.type, category: this.category, workTime: this.paras.workTime }).then(res => {
                this.loadcount--;
                var temp = res.data.map(e => {
                    e.qty = Number(e.qty);
                    return e;
                })
                const dv2 = new DataSet.View().source(temp)
                dv2
                    .transform({
                        type: 'percent',
                        field: 'qty',
                        dimension: 'name',
                        as: 'percent',
                    })
                    .transform({
                        type: 'sort-by',
                        fields: ['qty'], // 根据指定的字段集进行排序，与lodash的sortBy行为一致
                        order: 'DESC', // 默认为 ASC，DESC 则为逆序
                    })
                this.chartData = dv2.rows; 
            })
        },
        mounted() { },
        methods: {
        },
    }
</script>

<style lang="less">
    .ef-sfcs-stat-table-footer {
        font-weight: bold;
        background-color: @table-selected-row-bg;
    }
</style>
<style scoped lang="less">
    .title {
        font-weight: bold;
    }

    .basebg {
        background-color: @base-bg-color;
    }
</style>