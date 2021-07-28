<template>
    <div>
        <a-table class="basebg" :id="id" :size="size" :bordered="true" :columns="columns" :pagination="false"
            :data-source="tableData" :rowClassName="rowClassName" @change="onTableChange">
            <!--表頭title-->
            <span class="title" slot="noTitle">{{ $t('no') }}</span>
            <span class="title" slot="nameTitle">{{ $t(groupkey) }}</span>

            <!--序號欄位顯示-->
            <template slot="no" slot-scope="txt, rec, index"> {{index+1}}</template>
            <!--groupby顯示-->
            <template slot="name" slot-scope="txt, rec">
                <a-tooltip :title="txt">
                    <div style=" text-overflow: ellipsis;
                    overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                          <b>{{ txt || rec.key }}</b></div>
                </a-tooltip>
             </template>

            <!--rcs顯示-->
            <template v-for="rc in rcs" :slot="rc.rc" slot-scope="txt, rec">
                <span v-if=" rc.key!='' &&  handle_item(rec, rc.rc)[rc.rc].qty>0" :key="rc.key">
                    {{ handle_item(rec, rc.rc)[rc.rc].qty }}
                </span>
                <span :key="rc.key" v-else-if="rec.key != ''"> </span>
                <span :key="rc.key" v-else>{{ rec[rc.rc] }}</span>
            </template>

            
        </a-table>

        <v-dialog :group="group" :itemkey="itemkey" :size="size" :vshow="vshow" :params="values" :rc="rc"
            :rcname="rcname" :action-fix="actionFix">
        </v-dialog>
    </div>
</template>

<script>
    import bus from './bus.js'
    import VDialog from './dialog_component.vue'
    import { merge } from '@/utils/util'

    const preCols = [
        { key: 'no', align: 'center' },
        { key: 'name', align: 'left' },
    ].map((e) => {
        e.dataIndex = e.key
        e.slots = { title: e.key + 'Title' }
        e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
        return e
    })

    export default {
        name: "g5io_table",
        i18n: merge(require('@/i18n'), require('./i18n')),
        props: {
            actionFix: String,
            groupkey: String,
            size: String,
            id: String,
            data: Array,
            tabledata: Array,
            values: Object,
            rcnames: Object,
            group: String,
        },
        data() {
            return {
                sortOrder: '',
                sSize: 'middle',
                preCols,
                footerRowKey: '',
                itemkey: '',
                vshow: false,
                rc: '',
                rcname: '',
            }
        },
        components: {
            VDialog,
        },

        computed: {
            tableData() {
                var formdata = [];
                formdata = this.tabledata.reduce(function (cr, nx) {
                    const it = cr && cr.find((e) => e.key === nx.key)
                    it ?
                        (it[nx.rc] = nx) &
                        (it.name = it.name || nx.name)
                        : cr.push({
                            key: nx.key,
                            name: nx.name,
                            [nx.rc]: nx,
                        })
                    return cr
                }, [])
                var total = { key: "", name: "total" }
                formdata.map(e => {
                    this.rcs.map(a => {
                        total[a.rc] = Number(total[a.rc] ?? 0) + Number(e[a.rc] ? e[a.rc].qty : 0)
                    })
                })
                formdata.push(total)
                return formdata;
            },
            rcs() {
                var arr = [];
                for (var key in this.rcnames) {
                    arr.push({ rc: key, rcname: this.rcnames[key] })
                }
                return arr;
            },
            columns() {
                const cols = [];
                for (var key in this.rcnames) {
                    const col = {
                        key: key,
                        align: 'right',
                        ellipsis: 'true',
                        title: this.rcnames[key],
                        slots: { title: key + 'Title' },
                        scopedSlots: { customRender: key },
                    }
                    cols.push(col);
                }
                let rtn = [...preCols]
                rtn.splice(2, 0, ...cols)
                rtn = rtn.map((e) => {
                    e.sorter = e.key === 'no' ? false : (a, b) => this.sorter(a, b, e.key)
                    return e
                })
                return rtn
            },
        },
        methods: {
            rowClassName(record) {
                return record.key === this.footerRowKey ? 'ant-table-stat-footer' : null
            },
            onTableChange(pagination, filters, sorter) {
                this.sortOrder = sorter.order
            },

            getRowNo(index) {
                const len = (this.fpys ?? []).length
                return index === len ? '-' : index + 1
            },

            sorter(a, b, rcs) {
                const order = this.sortOrder
                const fKey = this.footerRowKey
                if (a.key === fKey) {
                    return order === 'ascend' ? 1 : -1
                } else if (b.key === fKey) {
                    return order !== 'ascend' ? 1 : -1
                } else if (rcs === 'name') {
                    const aN = a[rcs]
                    const bN = b[rcs]
                    return aN === bN ? 0 : aN > bN ? 1 : -1
                } else {
                    const aI = a[rcs].qty ?? 1
                    const bI = b[rcs].qty ?? 1
                    return Math.sign(aI - bI)
                }
            },

            handle_search(row, rc, rcname) {
                bus.$on('aavshow', (msg) => {
                    this.vshow = msg
                })
                this.rc = rc
                this.itemkey = row.key
                this.rcname = rcname
                this.vshow = true
            },

            handle_item(obj, element) {
                var res = Object.prototype.hasOwnProperty.call(Object(obj), element)
                var arr = obj
                if (!res) {
                    arr[element] = { qty: '0', key: arr.key }
                }
                return Object(arr)
            },
        },
    }
</script>

<style scoped lang="less">
    .basebg {
        background-color: @base-bg-color;
    }
</style>