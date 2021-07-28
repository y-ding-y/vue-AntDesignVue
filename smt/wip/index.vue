<template>
    <ef-main :id="imageId" :isMainPage="true" :title="$t('wipTitle')" :loading="loading" :openSubmit="true"
        :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" :i18n="i18n" @submit="submit"
        @resize="resize" @change="handle_change">

        <v-table :id="tableId" :size="sSize" :tabledata="tabledata" :column="table_column" :action-fix="'sfcsSMTWipWo'"
            :values="values">



        </v-table>
    </ef-main>
</template>

<script>
    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import { merge } from '@/utils/util' // saveToExcel
    import VTable from './table_component.vue'
    const DataSet = require('@antv/data-set')
    const formItems = [
    {  key: 'BU', type: 'select', paraKey: 'SfcsSmt.BU', value: "GPI", allowClear: false, },
    { key: 'class', type: 'select', paraKey: 'SfcsSmt.CLASS',value:"MB" ,allowClear: false,},
        {
            key: 'family',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsSmt.FAMILY'
        },
        {
            key: 'line',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsSmt.LINE'
        },
        {
            key: 'class',
            type: 'select',
            paraKey: 'SfcsSmt.CLASS'
        },
        {
            key: 'pn',
            type: 'popinput'
        },

        {
            key: 'wipwosite',
            type: 'select',
            paraKey: 'SfcsSmt.WIPWOSET',
        },
        {
            key: 'wo',
            type: 'input',
            value: "",
            style: "width:200px;max-width:500px",
        },
    ]

    const i18n = require('./i18n')
    export default {
        name: 'sfcsSMTWipWo',
        i18n: merge(require('@/i18n'), i18n),
        components: { EfMain, VTable },
        data() {
            return {
                i18n,
                imageId: 'defectimage' + new Date().format('hiu'),
                tableId: 'defecttable' + new Date().format('hiu'),
                loadcount: 0,
                formItems,
                sSize: 'middle',

                table_column: [],
                tabledata: [],

                total_target: 0,
                total_input: 0,

                values: {},

                woslist: [],
            }
        },
        computed: {
            footname() {
                return this.$t('ttl')
            },

            loading() {
                return this.loadcount > 0
            },

        },
        watch: {
            footname() {
                return this.tabledata.map((e) => {
                    if (e.key == "") {
                        e.wo = this.footname
                    }
                    return e;
                })
            },
        },
        created() {
            reqPost('SFCS_SMT', { key: "WIPWOSET", prop: "consts" }).then((res) => {
                this.woslist = res.data;
                this.loadcount -= 1
            })
            this.loadcount += 1
           // this.handle_wiprc();
        },
        mounted() { },
        methods: {
            handle_change(values) {
                this.woslist.some(e => {
                    if (e.key === values.wipwosite && e.wos) {
                        var arr = e.wos.replace(/\r\n/g, ",");
                        formItems[7].value = arr;
                        return true;
                    }
                    else {
                        formItems[7].value = "";
                    }
                })
            },
            submit(values) {
                console.log(values);
                this.values = values;
                this.handle_mrp(values);
            },

            resize(size) {
                this.sSize = size
            },

            //mrp
            handle_mrp(values) {
                var params = Object.assign({}, values);
                params.prop = "consts";
                params.key = "ORGIDS";
                console.log(params);
                reqPost('ERP_MRP', params).then((res) => {
                    // var temp_wo = [];
                    var temp_org = [];
                    for (var key in res.data) {
                        if (key != "581") {
                            temp_org.push(key)
                        }
                    }
                    // res.data.map(e => {

                    //     if (e.class == "SYS" || e.class == "FRYU") {
                    //         e.so = (!e.so ? "" : e.so + "-" + e.line)
                    //         e.scheduleshipdate = !e.scheduleshipdate ? e.scheduleshipdate : e.scheduleshipdate.substring(5, 10);
                    //         temp_wo.push(e)
                    //     }
                    // });
                    // this.handle_wo(temp_wo, values);
                    this.handle_mrp_wo(temp_org, values)
                    this.loadcount -= 1
                })
                this.loadcount += 1
            },

            handle_mrp_wo(element, values) {
                var params = Object.assign({}, values);
                params.prop = "wo";
                params.orgId = element;
                params.status = "3";
                params.select = "wo as WO";
                reqPost('ERP_MRP', params).then((res) => {
                    //  var temp_wo = [];
                    // res.data.map(e => {
                    //     if (e.class == "SYS" || e.class == "FRYU") {
                    //         e.so = (!e.so ? "" : e.so + "-" + e.line)
                    //         e.scheduleshipdate = !e.scheduleshipdate ? e.scheduleshipdate : e.scheduleshipdate.substring(5, 10);
                    //         temp_wo.push(e)
                    //     }
                    // });
                    this.handle_wo(res.data, values);
                    this.loadcount -= 1
                })
                this.loadcount += 1
            },

            //wo 
            handle_wo(temp_wo, values) {
                var params = Object.assign({}, values);
                params.prop = "wo";
                params.select = {
                    'WO': 'wo',"PN":"pn","PNDESC":"pnDesc", 'FAMILYNAME': 'familyName', 'TARGETQTY': 'targetQty', 'INPUT': 'inputQty',
                    'OUTPUT': 'outputQty',
                    //  'LINENAME': 'inputLineName',
                    'INPUTDAYS': 'inputDays'
                };
                params.wo = (formItems[7].value == "" ? temp_wo.map(e => {
                    return e.wo;
                }) : formItems[7].value)
                console.log(params);
                reqPost('SFCS_SMT', params).then((res) => {
                    var temp = []; this.total_input = 0; this.total_target = 0;
                    res.data.map(e => {
                        this.total_target = this.total_target + Number(e.targetqty);
                        this.total_input = this.total_input + Number(e.input);
                        temp_wo.some(a => {
                            if (e.wo === a.wo) {
                                e.inputdays = Number(e.inputdays);
                                e.uninput = Number(e.targetqty) - Number(e.input);
                                e.input = Number(e.input)
                                temp.push({ ...e, ...a })
                                return true
                            }
                        })
                    })
                    this.handle_sn(temp, values);
                    this.loadcount -= 1
                })
                this.loadcount += 1
            },

            //column
            // handle_wiprc() {
            //     var params = { prop: "consts", "key": "WIPRC", };
            //     reqPost('SFCS_SMT', params).then((res) => {
            //         this.table_column = res.data;
            //         this.loadcount -= 1
            //     })
            //     this.loadcount += 1
            // },

            //数组去重
            dedupe(array) {
                return Array.from(new Set(array));
            },

            handle_sn(temp_wo, values) {
                var params = Object.assign({}, values);
                params.prop = "sn";
                params.select = { 'qty': 'COUNT(*)' };
                params.group = ['wo', 'wipRouteCode'];//.push('wo');
                params.wo = (formItems[7].value == "" ? temp_wo.map(e => e.wo) : formItems[7].value);
                console.log(params);
                reqPost('SFCS_SMT', params).then((res) => {
                    var temp_cols = [];
                    var total = res.data.map(e => {
                        e.qty = Number(e.qty);
                        temp_cols.push(e.wiproutecode)
                        return e;
                    }).reduce(function (cr, nx) {
                        const it = cr && cr.find((e) => e.wiproutecode === nx.wiproutecode)
                        it ? (it.qty += nx.qty) & (it.wiproutecode = it.wiproutecode || nx.wiproutecode)
                            : cr.push({
                                qty: nx.qty,
                                wiproutecode: nx.wiproutecode,
                            })
                        return cr
                    }, [])
                    this.table_column = this.dedupe(temp_cols);
                    temp_wo.map(e => {
                        res.data.map(a => {
                            if (e.wo === a.wo) {
                                e[a.wiproutecode] = a.qty;
                            }
                        })
                        return e;
                    })


                    this.table_column.map(e => {
                        const dv = new DataSet.DataView().source(temp_wo).transform({
                            type: 'impute',
                            field: e, // 待补全字段
                            groupBy: [], // 分组字段集（传空则不分组）
                            method: 'value', // 补全字段值时执行的规则
                            value: 0
                        });
                        temp_wo = dv.rows;
                    })

                    //total
                    var temp_obj = {
                        key: '',
                        wo: this.footname,
                        targetqty: this.total_target,
                        input: this.total_input
                    };
                    total.map((e) => {
                        temp_obj[e.wiproutecode] = e.qty;
                    })
                    temp_wo.push(temp_obj)
                    this.tabledata = temp_wo; 
                    this.loadcount -= 1
                })
                this.loadcount += 1
            },
        },
    }
</script>

<style scoped lang="less">
    .title {
        font-weight: bold;
    }

    .basebg {
        background-color: @base-bg-color;
    }
</style>