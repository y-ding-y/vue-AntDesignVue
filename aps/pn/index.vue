<template>
    <ef-main :isMainPage="true" :dataTableId="tableId" :loading="loading" :form-items="formItems " :openSubmit="true"
        id="a111" mainPageId="a111" @submit="submit" @resize="resize">
        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :scroll="scroll"
            :pagination="false" :data-source="tabledata" rowKey="wo" :rowClassName="rowClassName">
            <template slot="no" slot-scope="txt, rec, index">
                <span v-if="!rec.isaction"> {{index+1}}</span>
                <span v-else> {{index+1}}</span>
            </template>


            <template v-for="item in configs" :slot="item.id" slot-scope="txt, rec">
                <span :key="item.id" v-if="rec[item.id]">
                    <a-tooltip :title="rec[item.id].desc">
                        <span>{{rec[item.id].pn}}</span>*
                        <b> {{rec[item.id].qty}}</b>
                    </a-tooltip>
                </span>

            </template>

            <template slot="wo" slot-scope="txt">
                <a @click="handle_wo(txt)">{{ txt }}</a>
            </template>

            <template slot="action" slot-scope="txt,rec">
                <a-button type="link" size="small" :class="'success'" @click="handle_refreshwo(rec)">
                    <a-icon type="sync" />
                </a-button>
            </template>
        </a-table>


    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'

    import { merge } from '@/utils/util'//,strHashColor,randomColor
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'

    const formItems = [
        { key: 'routeconfig', type: 'select', paraKey: 'APS.route', value: "3" },
        { key: 'family', type: 'select', mode: 'multiple', paraKey: 'APS.pjname', },
        { key: 'wo_pn', type: 'poptextarea' },
        { key: 'wo', type: 'poptextarea' },
        //工單 成品料號 route 機種多選 
    ]
    const i18n = require('./i18n')
    export default {
        name: 'workerInfo',
        components: { EfMain },
        i18n: merge(require('@/i18n'), i18n),

        data() {
            return {
                url: 'http://localhost:8081/apspn',
                tableId: 'oqaworkerdstation' + new Date().format('hiu'),
                loadcount: 0,
                tabledata: [],
                sSize: 'small',
                temp_rec: {},
                configs: [],
                formItems,
                colors: "85%",
                routeconfig: "",
                windowHeight: document.documentElement.clientHeight,
            };
        },
        watch: {
            theme() {
                return this.colors = this.theme.mode == "night" ? "25%" : "85%"
            },
        },
        // mounted() {
        //     var that = this;
        //     // <!--把window.onresize事件挂在到mounted函数上-->
        //     window.onresize = () => {
        //         return (() => {
        //             window.fullHeight = document.documentElement.clientHeight;
        //             that.windowHeight = window.fullHeight;  // 高 
        //         })()
        //     };
        // },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight', 'theme']),
            scroll() {
                return { x: this.configs.length * 160 + 500, y: this.pageMinHeight - 132 }
            },
            loading() {
                return this.loadcount > 0;
            },
            columns() {
                var arr = this.configs.map(e => {
                    e.dataIndex = e.id
                    e.align = "left"
                    e.dataIndex = e.id
                    e.title = e.name
                    e.slots = { title: e.name }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.id }
                    e.sorter = (a, b) => this.sorter(a, b, e.id)
                    e.customCell = (record) => {		//在此处可以修改单元格中的样式
                        return {
                            style:
                            {
                                'backgroundColor': record[e.id] ? ("hsl(" + record[e.id].color + "," + this.colors + ")") : "",
                                '-webkit-line-clamp': 2,
                            }
                        }    //return 想要设置的样式
                    }
                    return e
                })
                var temp = [
                    { key: 'no', align: 'center', fixed: "left", title: this.$t('no'), width: 50 },
                    { key: 'wo', align: 'left', fixed: "left", title: this.$t('wo'), width: 150 },
                    { key: 'wo_pn', align: 'left', fixed: "left", title: this.$t('wo_pn'), width: 120 },
                    { key: 'pj_name', align: 'left', title: this.$t('pj_name'), width: 100 },
                    { key: 'qty', align: 'right', title: this.$t('qty'), width: 70 },
                    { key: 'releasedate', align: 'left', title: this.$t('releasedate'), width: 100 },
                    { key: 'product', align: 'left', title: this.$t('product'), width: 80 },
                    //{ key: 'qty', align: 'right', title: this.$t("qty") },
                ].map((e) => {
                    e.dataIndex = e.key
                    e.sorter = ['no', 'action'].includes(e.key) ? false : (a, b) => this.sorter1(a, b, e.key)
                    e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e
                });
                return [...temp, ...arr, ...[
                    { key: 'action', align: 'center', title: this.$t('refresh'), width: 60 }].map((e) => {
                        e.dataIndex = e.key
                        e.slots = { title: this.$t(e.key) }
                        e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                        return e
                    })
                ]
            },

        },
        created() {
            // if (Object.values(this.$route.query).length > 0) {
            //     console.log(this.$route.query)
            // }
            // else {
            //     //
            // }
            // this.handle_search();
        },
        methods: {
            //数组去重
            dedupe(array) {
                return Array.from(new Set(array));
            },

            submit(values) {
                var temp_values = { ...values };
                temp_values.wo = this.setNewVal(values.wo)
                temp_values.wo_pn = this.setNewVal(values.wo_pn)
                this.routeconfig = temp_values.routeconfig;
                this.handle_search(temp_values)
            },
            handle_search(values) {
                this.loadcount = 3;
                var params = {
                    prop: "getapspn",
                }
                reqPost('APS', { ...params, ...values }).then(res => {
                    this.loadcount--;
                    var pn = res.data;

                    var temp_pn = this.dedupe(res.data.map(e => {
                        return e.config
                    }))
                    var temp = temp_pn.map(e => {
                        var tempe = {};
                        tempe.config = e;
                        tempe.pn = [];
                        pn.map(a => {
                            if (e == a.config) {
                                tempe.pn.push(a);
                            }
                        })
                        return tempe;
                    })
                    var color_pn = [];
                    temp.map((e, i) => {
                        var temp_2 = this.dedupe(e.pn.map(m => {
                            return m.pn
                        }))
                        var cou = 360 / Number(temp_2.length)
                        temp_2.map((o, index) => {
                            e.pn.map(a => {
                                if (a.pn == o) {
                                    color_pn.push({ ...a, ...{ h: (index + 1) * cou, s: (i + 1) * cou } })
                                }
                            })
                        })

                    })
                    reqPost('APS', { prop: "getapsconfig", route: values.routeconfig }).then(res2 => {
                        this.loadcount--;
                        this.configs = res2.data;
                        reqPost('APS', { prop: "getapswo", ...values }).then(res1 => {
                            this.loadcount--;
                            this.tabledata = res1.data.map(e => {
                                e.releasedate = e.released_date.slice(0, 10);
                                color_pn.some(a => {
                                    if (e.wo == a.wo) {
                                        e[a.config] = {};
                                        e[a.config].pn = a.pn;
                                        e[a.config].qty = a.qty;
                                        e[a.config].desc = a.description;
                                        e[a.config].color = a.h + ", " + a.s + "%";
                                    }
                                })
                                return e;
                            })

                        })
                    })
                })
            },
            sorter(a, b, key) {
                const aN = a[key] ? a[key].pn : ""
                const bN = b[key] ? b[key].pn : ""
                return aN === bN ? 0 : aN < bN ? 1 : -1
            },
            sorter1(a, b, key) {
                if (key == "qty") {
                    const aN = a[key] ?? 0
                    const bN = b[key] ?? 0
                    return Math.sign(aN - bN)
                }
                else {
                    const aN = a[key] ?? ""
                    const bN = b[key] ?? ""
                    return aN === bN ? 0 : aN < bN ? 1 : -1
                }


            },

            rowClass(record) {
                return record.key === this.footerRowKey ? 'table-footer' : null
            },
            handle_wo(e) {
                window.open("/gtk/material/wo?wo=" + e, '_blank');
            },

            handle_refreshwo(e) {
                var params = {
                    prop: "refreshwo",
                    wo: e.wo,
                    org_id: e.org_id,
                    deptno: e.deptno,
                    product: e.product,
                    action: "refreshwo",
                    routeconfig: this.routeconfig,
                };
                this.loadcount = 1;
                reqPost('APS', params).then(res => {
                    this.loadcount--;
                    // var temp = { ...e };
                    var temp = {
                        deptno: e.deptno,
                        id: e.id,
                        org_id: e.org_id,
                        pj_name: e.pj_name,
                        product: e.product,
                        qty: e.qty,
                        released_date: e.released_date,
                        releasedate: e.releasedate,
                        status: e.status,
                        wo: e.wo,
                        wo_pn: e.wo_pn,
                    };
                    if (res.data.length > 0) {
                        res.data.map(a => {
                            temp[a.config] = {};
                            temp[a.config].desc = a.description;
                            temp[a.config].pn = a.pn;
                            temp[a.config].qty = a.qty;
                        })
                        var index = this.tabledata.findIndex(function (value) {
                            return value.wo == temp.wo
                        })
                        console.log(index)
                        this.tabledata.splice(index, 1, temp);
                    }
                })
            },

            rowClassName(record, index) {
                const className = this.theme.mode == 'night' ? 'dark' : 'light';
                return index % 2 === 1 ? className : '';
            },

            setNewVal(e) {
                var res = [];
                if (e && e != "") {
                    const pattern = /[`~!@#$^\-&*()=|{}':;',<>?~！@#￥……&*（）——|{}【】'；：""'。，、？\s]/g
                    e.replace(pattern, ",").split(',').map(o => {
                        if (o != "") {
                            res.push(o)
                        }
                    })
                }
                return res
            },

            resize(size) {
                this.sSize = size
            },

        },
    };
</script>


<style scoped lang="less">
    .title {
        font-weight: bold;
    }

    .basebg {
        background-color: @base-bg-color;
    }

    .success,
    .success:hover,
    .success:visited {
        /* background-color: @success-color; */
        color: @success-color;
    }

    .warning,
    .warning:hover,
    .warning:visited {
        /* background-color: @success-color; */
        color: @warning-color;
    }
</style>

<style lang="less">
    .ant-table .light {
        background-color: @primary-1;
    }

    .ant-table .dark {
        background-color: @shadow-color;
    }

    /*     
    .ant-table .light {
        background-color: white !important;
    }

    .ant-table .dark {
        background-color: #E3EFF6 !important;
    } */
</style>