<template>
    <div>
        <a-modal v-model="visible" :title="title" :width="listWidth" :footer="null">
            <div style="text-align:right;margin-top:-20px;margin-bottom:5px;">
                <a-icon type="file-excel" @click="onSaveList" />
            </div>
            <a-table class="basebg" :id="listId" :size="'small'" :bordered="true" :columns="listCols"
                :pagination="false" :data-source="listData" :loading="listLoading" rowKey="sn"
                :scroll="{ y: 600, x: listTableWidth }">
                <template slot="no" slot-scope="txt, rec, index">{{ index+1 }} </template>

                <template slot="sn" slot-scope="txt">
                    <a @click="handle_sn(txt)">{{txt}}</a>
                </template>
            </a-table>
        </a-modal>

    </div>
</template>

<script>
    import { reqPost } from '@/services/base'
    import { merge, saveToExcel } from '@/utils/util'


    const sorter = (a, b, key) => (a[key] === b[key] ? 0 : a[key] > b[key] ? 1 : -1)

    export default {
        name: 'g5wip_dialog',
        i18n: merge(require('@/i18n'), require('./i18n')),
        props: {
            actionFix: String,
            vkey: { type: Number, default: 0 },
            paras: { type: Object, default: () => ({}) },
            prop: { type: String, default: "" },
            title: { type: String, default: "List" },
        },
        data() {
            return {
                loadcount: 0,
                visible: false,
                listTableWidth: 1300,
                listWidth:
                    document.documentElement.clientWidth * 0.8 > 1000
                        ? 1000
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
                    { key: 'no', align: 'center', width: 50 },
                    { key: 'sn', align: 'left', width: 150 },
                    { key: 'familyname', width: 100 },
                    { key: 'pn', width: 150 },
                    { key: 'wo', width: 130 },
                    { key: 'linename', width: 120 },
                    { key: 'lastroutecodename', width: 150 },
                    { key: 'wiproutecodename', width: 150 },
                    { key: 'statusname', width: 120 },
                    { key: 'time' },
                ].map((e) => {
                    e.title = this.$t(e.key);
                    e.dataIndex = e.key.toLowerCase()
                    e.slots = { title: e.key + 'T', customRender: e.key }
                    e.scopedSlots = e.slots
                    e.align = e.align || 'left'
                    e.key !== 'no' ? (e.sorter = (a, b) => sorter(a, b, e.dataIndex)) : ''
                    return e
                });
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
                this.paras.select = {
                    "SN": "sn", "PN": "pn", "WO": "wo", "FAMILYNAME": "familyName",
                    "LINENAME": "lineName", "LASTROUTECODENAME": "lastRouteCodeName",
                    "WIPROUTECODENAME": "wipRouteCodeName", "STATUSNAME": "statusName", "TIME": "time"
                }

                // "sn,pn,wo,lineName as NAME,lastRouteCodeName,wipRouteCodeName,statusName,time";
                reqPost('SFCS_SMT', this.paras).then((rec) => {
                    this.listLoading = false;
                    this.listData = rec.data.map(e => {
                        // e.familyname = this.paras.familyname;
                        // e.linename = this.paras.linename;
                        // e.wo = this.paras.wo;
                        return e;
                    });
                })
            },
            onSaveList() {
                saveToExcel(this.listId)
            },

            handle_sn(e) {
                //    let routeData = this.$router.resolve({ path: '/test' });
                //     window.open("http://localhost:8080/g5/product/snview?sn=" + e, '_blank');
                //  this.$router.push({ path: "../../../sys/product/sn/sys", query: { sn: e } });

                window.open("../../../smt/product/snview?sn=" + e, '_blank');

                // console.log(this.$rout er);
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