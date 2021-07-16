<template>
    <div>
        <a-modal v-model="visible" :title="title" :width="listWidth" :footer="null">
            <div style="text-align:right;margin-top:-20px;margin-bottom:5px;">
                <a-icon type="file-excel" @click="onSaveList" />
            </div>
            <a-table class="basebg" :id="listId" :size="'small'" :bordered="true" :columns="listCols"
                :pagination="false" :data-source="listData" :loading="listLoading"
                :scroll="{ y: 600, x: listTableWidth }">

                <template slot="ccmail" slot-scope="text, record">
                    <a-tooltip :title="record.ccmail">
                        <div style=" text-overflow: ellipsis;
                                overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                            {{ record.ccmail }}
                        </div>
                    </a-tooltip>
                </template>

                <template slot="tomail" slot-scope="text, record">
                    <a-tooltip :title="record.tomail">
                        <div style=" text-overflow: ellipsis;
                                overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                            {{ record.tomail }}
                        </div>
                    </a-tooltip>
                </template>

                <template slot="bcmail" slot-scope="text, record">
                    <a-tooltip :title="record.bcmail">
                        <div style=" text-overflow: ellipsis;
                                overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                            {{ record.bcmail }}
                        </div>
                    </a-tooltip>
                </template>

            </a-table>
        </a-modal>
    </div>
</template>

<script>
    import { reqPost } from '@/services/base'

    import { merge, saveToExcel } from '@/utils/util'

    export default {
        name: 'Detail_list',
        props: {
            actionFix: String,
            vkey: { type: Number, default: 0 },
            paras: { type: Object, default: () => ({}) },
            prop: { type: String, default: "" },
            title: { type: String, default: "List" },
        },

        i18n: merge(require('@/i18n'), require('./i18n')),
        data() {
            return {
                loadcount: 0,
                visible: false,
                listTableWidth: 2800,
                listWidth:
                    document.documentElement.clientWidth * 0.8 > 2000
                        ? 2000
                        : document.documentElement.clientWidth * 0.8,
                listLoading: false,
                listId: 'defectlists' + new Date().format('hiu'),
                listData: [],

            }
        },
        computed: {
            loading() {
                return this.loadcount > 0
            },
            listCols() {
                var arr = [
                    { key: 'seq', fixed: 'left', align: 'left', width: 80 },
                    { key: 'datatype', width: 150, fixed: 'left', },
                    { key: 'detaildatatype', width: 150 },
                    { key: 'getcode', width: 200 },
                    { key: 'coderadix', width: 120 },
                    { key: 'verificationmode', width: 110 },
                    { key: 'unit', width: 100 },
                    { key: 'showmsessage', width: 160 },
                    { key: 'showmin', align: 'right', width: 130 },
                    { key: 'showmax', align: 'right', width: 130 },
                    { key: 'controlmin', align: 'right', width: 130 },
                    { key: 'controlmax', align: 'right', width: 130 },
                    { key: 'enable', width: 80 },
                    { key: 'alarmflag', width: 80 },
                    { key: 'needwechat', width: 80 },
                    { key: 'needmail', width: 80 },
                    { key: 'tomail', width: 150 },
                    { key: 'ccmail', width: 150 },
                    { key: 'bccmail', width: 180 },
                    { key: 'updatetime', width: 180 },
                    { key: 'createtime', },
                ].map((e) => {
                    e.dataIndex = e.key
                    e.title = this.$t(e.key)
                    e.slots = { title: e.key + 'Title' }
                    e.scopedSlots = { customRender: e.slot ?? e.key }
                    e.sorter = e.key === 'no' ? false : (a, b) => this.sorter(a, b, e.key)
                    return e
                })
                return arr;
            },
        },
        watch: {
            vkey() {

                this.visible = true;
                this.onOpenList();
            },
        },
        created() {
        },
        mounted() { },
        methods: {
            onOpenList() {
                this.paras.prop = this.prop;
                this.listLoading = true;
                reqPost('SFCS_PP', this.paras).then((rec) => {
                    this.listLoading = false;
                    this.listData = rec.data.map((e, i) => {
                        e.key = i
                        return e;
                    })
                })
            },
            sorter(a, b, rcs) {
                if (['showmin', 'showmax', 'controlmin', 'controlmax'].includes(rcs)) {
                    const aI = a[rcs] ?? 0
                    const bI = b[rcs] ?? 0
                    return Math.sign(aI - bI)
                } else {
                    const aN = a[rcs] || ""
                    const bN = b[rcs] || ""
                    return aN === bN ? 0 : aN > bN ? 1 : -1
                }
            },
            onSaveList() {
                saveToExcel(this.listId)
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