<template>
    <div>
        <a-table class="basebg" :id="id" :size="size" :bordered="true" :columns="columns" :pagination="false"
            rowKey="wo" :data-source="tabledata" :rowClassName="rowClassName" @change="onTableChange" :scroll="scroll">
            <template slot="no" slot-scope="txt, rec, index">{{
                getRowNo(index)
                }}
            </template>

            <template v-for="item in column" :slot="item +'Title'">
                <span :key="item"><b>{{handle_rcname(item)}} </b> </span>
            </template>

            <template slot="pn" slot-scope="txt,rec ">
                <a-tooltip :title="rec.pndesc">
                    <span> {{txt}}</span>
                </a-tooltip>
            </template>


            <template v-for="(item,index) in ['uninput','input','inputdays']" :slot="item" slot-scope="txt ">
                <span v-if="txt>0" :key="item+(index++)"> {{txt}}</span>
            </template>



            <template slot="mtlqty" slot-scope="txt ">
                <span v-if="txt>0"> {{Number(txt).toFixed(1)}}</span>
            </template>

            <template v-for="item in column" :slot="item" slot-scope="txt, rec ">
                <a @click="handle_click(rec,item)" :key="item" v-if="txt>0">{{txt}}</a>
            </template>

            <template slot="customername" slot-scope="text, record">
                <a-tooltip :title="record.customername">
                    <div style=" text-overflow: ellipsis;
                    overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                        {{ record.customername }}</div>
                </a-tooltip>
            </template>

        </a-table>

        <v-dialog :vkey="key" :paras="paras" :prop="'sn'" :title="title" :action-fix="actionFix"></v-dialog>
    </div>
</template>

<script>
    import { merge } from '@/utils/util'
    import VDialog from './dialog_component.vue'
    import { reqPost } from '@/services/base'
    import { mapState } from 'vuex'

    export default {
        name: "g5wip_table",
        i18n: merge(require('@/i18n'), require('./i18n')),
        props: {
            actionFix: String,
            size: String,
            id: String,
            tabledata: Array,
            column: Array,
            values: Object,
        },
        data() {
            return {
                sortOrder: '',
                sSize: 'middle',
                footerRowKey: '',
                itemkey: '',
                vshow: false,
                rc: '',
                rcname: '',
                rcs: [],

                key: 0,
                paras: {},
            }
        },
        components: {
            VDialog,
        },
        created() {
            this.get_rcname();

        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { x: this.column.length * 140 + 840, y: this.pageMinHeight - 200 }
            },
            wos() {
                return this.tabledata.map(e => {
                    return e.wo;
                })
            },
            columns() {
                var per = [
                    { key: 'no', align: 'center', fixed: "left", width: 60 },
                    { key: 'wo', align: 'left', fixed: "left", width: 140 },
                    { key: 'pn', align: 'left', fixed: "left", width: 120 },
                    { key: 'familyname', align: 'left', width: 120 },
                    //    { key: 'so', align: 'left', width: 140 },
                    //     { key: 'salesname', align: 'left', width: 130 },
                    //  { key: 'customername', align: 'left', width: 100 },
                    //    { key: 'scheduleshipdate', align: 'center', width: 100 },
                    //    { key: 'linename', align: 'left', width: 90 },
                    { key: 'inputdays', align: 'right', width: 100 },
                    //  { key: 'mtlqty', align: 'right', width: 80 },
                    { key: 'targetqty', align: 'right', width: 90 },
                    // { key: 'input', align: 'right', width: 90 },
                    //  { key: 'uninput', align: 'right', width: 90 },
                ].map((e) => {
                    e.dataIndex = e.key;
                    e.title = this.$t(e.key);
                    e.slots = { title: e.key, customRender: e.key };
                    (e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }),
                        (e.sorter = e.key === 'no' ? false : (a, b) => this.sorter(a, b, e.key))
                    return e
                })
                var temp_col = [];
                this.rcs.map(e => {
                    this.column.some(a => {
                        if (a == e.key) {
                            temp_col.push(a)
                            return true;
                        }
                    })
                })
                var cols = temp_col.map(e => {
                    return {
                        key: e,
                        align: "right",
                        dataIndex: e,
                        slots: { title: e + "Title" },
                        scopedSlots: { customRender: e.slot ?? e },
                        sorter: (a, b) => this.sorter(a, b, e)
                    }
                })

                return [...per, ...cols]
            },
            title() {
                return this.$t('title')
            },
        },

        methods: {
            rowClassName(record) {
                return record.key === this.footerRowKey
                    ? 'ef-sfcs-stat-table-footer'
                    : null
            },

            getRowNo(index) {
                const len = (this.tabledata ?? []).length
                return index === len - 1 ? '-' : index + 1
            },

            onTableChange(pagination, filters, sorter) {
                this.sortOrder = sorter.order
            },
            sorter(a, b, rcs) {
                const order = this.sortOrder
                const fKey = this.footerRowKey
                if (a.key === fKey) {
                    return order === 'ascend' ? 1 : -1
                } else if (b.key === fKey) {
                    return order !== 'ascend' ? 1 : -1
                } else if (["wo", "pn", "familyname", "scheduleshipdate", "linename"].includes(rcs)) {
                    const aN = a[rcs] ?? ""
                    const bN = b[rcs] ?? ""
                    return aN === bN ? 0 : aN > bN ? 1 : -1
                } else {
                    const aI = a[rcs] ?? 1
                    const bI = b[rcs] ?? 1
                    return Math.sign(aI - bI)
                }
            },



            get_rcname() {
                reqPost('SFCS_SMT', { prop: "consts", key: "WIPRC" }).then((res) => {
                    this.rcs = res.data;
                })
            },

            handle_rcname(rc) {
                var res = "";
                this.rcs.some(a => {
                    if (a.key == rc) {
                        res = a.name
                        return true;
                    }
                })
                return res;
            },

            handle_click(e, rc) {
                this.key++
                const paras = {
                    ...this.values,
                    ...{ wipRouteCode: rc, wo: e.key == "" ? this.wos : e.wo, },
                }
                this.paras = paras
                // let routeData = this.$router.resolve({ path: '/test' });
                // window.open(routeData.href, '_blank'); 
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
    .basebg {
        background-color: @base-bg-color;
    }
</style>