<template>
    <div>
        <a-modal v-model="visible" :title="title" :width="listWidth" :footer="null">
            <div style="text-align:right;margin-top:-20px;margin-bottom:5px;">
                <a-icon type="file-excel" @click="onSaveList" />
            </div>
            <a-table class="basebg" :id="listId" :size="'small'" :bordered="true" :columns="listCols"
                :pagination="false" :data-source="listData" :loading="listLoading"
                :scroll="{ y: 600, x: listTableWidth }" rowKey="rowno">
                <span slot="noT" class="title">{{ $t('sn') }}</span>

                <template slot="no" slot-scope="t, r, i">{{ i + 1 }}</template>
                <template slot="sn" slot-scope="t">
                    <a @click="handle_sn(t)">{{ t }}</a>
                </template>

                <template slot="repairName" slot-scope="text, record">
                    <a-tooltip :title="record.repairname">
                        <div style=" text-overflow: ellipsis;
                        overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                            {{ record.repairname }}</div>
                    </a-tooltip>
                </template>
            </a-table>
        </a-modal>
    </div>
</template>

<script>
    import { reqPost } from '@/services/base'

    import { merge, saveToExcel } from '@/utils/util'

    const sorter = (a, b, key) => (a[key] === b[key] ? 0 : a[key] > b[key] ? 1 : -1)


    const i18n = require('./i18n')
    export default {
        name: 'Ship_list',
        props: {
            actionFix: { type: String, default: 'sfcsG5Ship' },
            vkey: { type: Number, default: 0 },
            paras: { type: Object, default: () => ({}) },
            prop: { type: String, default: "" },
            title: { type: String, default: "List" },
        },
        i18n: merge(require('@/i18n'), i18n),

        data() {
            return {
                i18n,
                loadcount: 0,
                visible: false,
                listTableWidth: 1600,
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
                return [
                    { key: 'no', align: 'center', width: 30 },
                    { key: 'bu', width: 40 },
                    { key: 'shipdate', width: 120 },
                    { key: 'customer', width: 150 },
                    { key: 'gg', width: 80 },
                    { key: 'modelname', width: 60 },
                    { key: 'class', width: 50 },
                    { key: 'pn', width: 60 },
                    { key: 'qty', width: 50, align: "right" },
                    { key: 'weight', width: 50, align: "right" },
                    { key: 'palletno', width: 120 },
                    // { key: 'plant', align: 'left', width: 110 },
                    // { key: 'packingno', width: 90 }, 

                ].map((e) => {
                    e.dataIndex = e.key.toLowerCase()
                    e.title = this.$t(e.key)
                    e.slots = { customRender: e.key }
                    e.scopedSlots = e.slots
                    e.align = e.align || 'left'
                    e.key !== 'no' ? (e.sorter = (a, b) => sorter(a, b, e.dataIndex)) : ''
                    return e
                })
            },
        },
        watch: {
            vkey() {
                this.visible = true;
                this.listData = [];
                this.onOpenList();
            },
        },
        created() {
        },
        mounted() { },
        methods: {
            onOpenList() {
                this.paras.prop = this.prop;
                console.log(this.paras);
                this.listLoading = true;
                reqPost("SFCS_G5", this.paras)
                    .then(res => {
                        this.listLoading = false;
                        console.log(res.data)
                        this.listData = res.data;
                    })
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