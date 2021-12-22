<template>
    <ef-main :id="imageId" :isMainPage="true" :loading="loading" :dataTableId="tableId" :imageDomId="imageId"
        @resize="resize" :goback="true" :key="id">
        <ef-chart class="basebg" :height="chartHeight" :padding="[44, 64, 48, 64]" :scale="[
         { dataKey: 'rate',  formatter: '.0%', min: 0 },
         { dataKey: 'qty',  tickInterval:5  },
      ]" :series="[
        {
          quickType: 'bar',
          position: 'name*qty', 
          adjust: [
            {
              type: 'dodge',
              marginRatio: 1 / 32,
            },
          ],
        },
        {
            quickType: 'line',
            position: 'name*rate',
            color:['type',['#a200ff']]
          },
          {
            quickType: 'point',
            position: 'name*rate',
            label:'rate', 
            color:['type',['#a200ff']]
          },

      ]" :data="chartData" @click_interval="click_interval">
        </ef-chart>
        <!-- <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
            :data-source="tableData" :rowClassName="rowClassName" @change="onTableChange">
            
            <template slot="no" slot-scope="txt, rec, index">{{
                index+1
                }}</template>
            
            <template slot="name" slot-scope="txt"><b>{{ txt }}</b></template>
           
            <template slot="qty" slot-scope="txt"><span v-if="txt && txt > 0">{{ txt }}</span></template>

            <template slot="qty" slot-scope="txt,rec">
                <a v-if="txt!=0" @click="handle_click(rec)">{{ txt }}</a>
            </template>
        </a-table> -->

        <defect-list :vkey="key" :temptable="temptable" :deptid="deptid" :title="title" :action-fix="'annuChart'"
            :dept="$props.dept">
        </defect-list>

    </ef-main>
</template>

<script>
    // import DataSet from '@antv/data-set'
    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import Chart from '@/components/chart/Chart'
    import { merge } from '@/utils/util'
    import DefectList from '@/pages/worker/annuchart/detailList'
    import { mapState } from 'vuex'

    // const formItems = [
    //     { key: 'dept', type: 'select', paraKey: "WORKER.dept", allowClear: false, },
    // ]

    const namelist = [{ key: "1", name: "0.5年以下", qty: 0 },
    { key: "2", name: "0.5-1年", qty: 0 },
    { key: "3", name: "1-2年", qty: 0 },
    { key: "4", name: "2-5年", qty: 0 },
    { key: "5", name: "5-10年", qty: 0 },
    { key: "6", name: "10年以上", qty: 0 },]
    export default {
        name: 'annuChart',
        i18n: merge(require('@/i18n'), require('./i18n')),
        components: { EfMain, EfChart: Chart, DefectList },
        props: {
            dept: { type: String, default: "" },
        },
        data() {
            return {
                imageId: 'fpyimage' + new Date().format('hiu'),
                tableId: 'fpytable' + new Date().format('hiu'),
                loadcount: 0,
                sSize: 'middle',
                //   formItems,
                rows: [],
                sortOrder: '',
                footerRowKey: '',

                key: 0,
                paras: {},
                values: {},
                rcs: [],
                chartData: [],
                datalist: [],
                temptable: [],
                deptlist: [],
                deptname: "",
                forms: {},
                deptid: "",
                id: 0,
            }
        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            title() {
                return this.$t('title')
            },
            loading() {
                return this.loadcount > 0
            },
            chartHeight() {
                return this.pageMinHeight - 200
            },

            tableData() {
                return this.chartData
            },
            columns() {
                return [
                    { key: 'no', align: 'center', title: this.$t('no'), width: 100 },
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100 },
                    { key: 'qty', align: 'right', title: this.$t('qty'), width: 100 },
                ].map((e) => {
                    e.dataIndex = e.key
                    e.sorter = ['no', 'name'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e
                })
            },
        },
        watch: {
            dept() {
                this.id++;
            },
        },
        created() {
            this.loadcount = 1;
            reqPost('WORKER', { prop: "getdeptid", name: this.$props.dept }).then(res => {
                this.loadcount--;
                this.deptid = res.data;
                this.handle_search();
            })
        },
        mounted() { },
        methods: {
            submit(values) {
                this.values = values;
                this.forms = { ...values }
                this.handle_search()
            },
            resize(size) {
                this.sSize = size
            },
            onTableChange(pagination, filters, sorter) {
                this.sortOrder = sorter.order
            },
            /**
             * 請求直通率數據
             */
            handle_search() {
                this.loadcount += 1
                reqPost('WORKER', { prop: "getannuchart", detp: this.deptid }).then((res) => {
                    this.loadcount -= 1
                    this.datalist = res.data;
                    var total = res.data.length;
                    var arr = [];
                    for (var i = 0; i < 6; i++) {
                        arr.push({ qty: 0, key: i + 1, name: namelist[i].name })
                    }

                    res.data.map(e => {
                        if (Number(e.annu) < 0.5) {
                            arr[0].qty = arr[0].qty + 1;
                        }
                        else if (Number(e.annu) >= 0.5 && Number(e.annu) < 1) {
                            arr[1].qty = arr[1].qty + 1;
                        }
                        else if (Number(e.annu) >= 1 && Number(e.annu) < 2) {
                            arr[2].qty = arr[2].qty + 1;
                        }
                        else if (Number(e.annu) >= 2 && Number(e.annu) < 5) {
                            arr[3].qty = arr[3].qty + 1;
                        }
                        else if (Number(e.annu) >= 5 && Number(e.annu) < 10) {
                            arr[4].qty = arr[4].qty + 1;
                        }
                        else if (Number(e.annu) >= 10) {
                            arr[5].qty = arr[5].qty + 1;
                        }
                    })
                    arr.map(e => {
                        e.type = "占比";
                        e.rate = Number((e.qty / total).toFixed(2));
                        return e;
                    })
                    this.chartData = arr;
                    console.log(this.chartData)

                })
            },

            rowClass(record) {
                return record.key === this.footerRowKey ? 'table-footer' : null
            },
            /**
             * 排序算法
             * 可保證匯總行(key為footerRowKey始終保持在表格最低部)
             */
            sorter(a, b, rcs) {
                const order = this.sortOrder
                const fKey = this.footerRowKey
                if (a.key === fKey) {
                    return order === 'ascend' ? 1 : -1
                } else if (b.key === fKey) {
                    return order !== 'ascend' ? 1 : -1
                } else if (rcs === 'rcname' || rcs === 'rate') {
                    const aN = a[rcs]
                    const bN = b[rcs]
                    return aN === bN ? 0 : aN > bN ? 1 : -1
                } else {
                    const aI = a[rcs] ?? 0
                    const bI = b[rcs] ?? 0
                    return Math.sign(aI - bI)
                }
            },
            getRowNo(index) {
                const len = (this.rows ?? []).length
                return index === len ? '-' : index + 1
            },
            rowClassName(record) {
                return record.key === this.footerRowKey ? 'ef-sfcs-stat-table-footer' : null
            },

            handle_click(e) {
                this.key++
                this.visible = true
                this.listLoading = true
                var temptable = [];
                switch (e.key) {
                    case 1:
                        this.datalist.map(e => {
                            if (Number(e.annu) < 0.5) {
                                temptable.push(e)
                            }
                        })
                        break;
                    case 2:
                        this.datalist.map(e => {
                            if (Number(e.annu) >= 0.5 && Number(e.annu) < 1) {
                                temptable.push(e)
                            }
                        })
                        break;
                    case 3:
                        this.datalist.map(e => {
                            if (Number(e.annu) >= 1 && Number(e.annu) < 2) {
                                temptable.push(e)
                            }
                        })
                        break;
                    case 4:
                        this.datalist.map(e => {
                            if (Number(e.annu) >= 2 && Number(e.annu) < 5) {
                                temptable.push(e)
                            }
                        })
                        break;
                    case 5:
                        this.datalist.map(e => {
                            if (Number(e.annu) >= 5 && Number(e.annu) < 10) {
                                temptable.push(e)
                            }
                        })
                        break;
                    case 6:
                        this.datalist.map(e => {
                            if (Number(e.annu) >= 10) {
                                temptable.push(e)
                            }
                        })
                        break;
                    default:
                        break;
                }
                this.temptable = temptable;

            },
            click_interval(e) {
                var _this = this;
                e._origin ?
                    (function () {
                        _this.handle_click(e._origin);
                    }())
                    :
                    (function () {
                        alert("請點擊柱狀圖")
                    }())
            },
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