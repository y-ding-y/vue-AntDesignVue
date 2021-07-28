<template>
    <div>
        <a-modal v-model="visible" :title="title" :width="listWidth" :footer="null">
            <div style="text-align:right;margin-top:-20px;margin-bottom:5px;">
                <a-icon type="file-excel" @click="onSaveList" />
            </div>
            <a-table class="basebg" :id="listId" :size="'small'" :bordered="true" :columns="listCols"
                :pagination="false" :data-source="listData" :loading="listLoading"
                :scroll="{ y: 600, x: listTableWidth }">
                <span slot="noT" class="title">{{ $t('sn') }}</span>
                <span slot="snT" class="title">{{ $t('productSn') }}</span>
                <span v-for="e in 'bu,class,modelName,pn,wo,lineName,routeCodeName,defectName,workerName,itime,rootName,repairName,dutyName,repairerName,otime'.split(
              ','
            )" :key="e" class="title" :slot="e + 'T'">{{ $t(e) }}</span>
                <template slot="no" slot-scope="t, r, i">{{ i + 1 }}</template>
                <template slot="sn" slot-scope="t">
                    <a @click="handle_sn(t)">{{ t }}</a>
                </template>

                <template slot="modelName" slot-scope="text, record">
                    <a-tooltip :title="record.modelname">
                        <div style=" text-overflow: ellipsis;
                        overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                            {{ record.modelname }}</div>
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
    const listCols = [
        { key: 'no', align: 'center', width: 50 },
        { key: 'sn', align: 'left', width: 150 },
        { key: 'bu', width: 60 },
        { key: 'class', width: 90 },
        { key: 'modelName', width: 150 },
        { key: 'pn', width: 130 },
        { key: 'wo', width: 120 },
        { key: 'lineName', width: 100 },
        { key: 'routeCodeName', width: 160 },
        { key: 'defectName', width: 160 },
        { key: 'workerName', width: 80 },
        { key: 'itime', width: 150 },
        { key: 'repairName', width: 200 },
        { key: 'rootName' },
        { key: 'dutyName', width: 120 },
        { key: 'repairerName', width: 80 },
        { key: 'otime', width: 150 },
    ].map((e) => {
        e.dataIndex = e.key.toLowerCase()
        e.slots = { title: e.key + 'T', customRender: e.key }
        e.scopedSlots = e.slots
        e.align = e.align || 'left'
        e.key !== 'no' ? (e.sorter = (a, b) => sorter(a, b, e.dataIndex)) : ''
        return e
    })

    const i18n = require('./i18n')
    export default {
        name: 'g5Defect_list',
        props: {
            actionFix: String,
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
                listTableWidth: 2100,
                listWidth:
                    document.documentElement.clientWidth * 0.8 > 2000
                        ? 2000
                        : document.documentElement.clientWidth * 0.8,
                listLoading: false,
                listId: 'defectlists' + new Date().format('hiu'),
                listData: [],
                listCols,
            }
        },
        computed: {
            loading() {
                return this.loadcount > 0
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
                reqPost('SFCS_SMT', this.paras).then((rec) => {
                    this.listLoading = false;
                    this.listData = rec.data.map((e, i) => {
                        e.key = i
                        return e;
                    })
                })
            },
            onSaveList() {
                saveToExcel(this.listId)
            },

            handle_sn(e) {
                window.open("../../../g5/product/snview?sn=" + e, '_blank');
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