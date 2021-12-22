<template>
    <div>
        <a-modal v-model="visible" :title="title" :width="listWidth" :footer="null">
            <a-table class="basebg" :id="listId" :size="'small'" :bordered="true" :columns="listCols"
                :pagination="false" rowKey="empno" :data-source="temptable" :loading="listLoading" :scroll="{ y: 600 }">
                <template slot="no" slot-scope="txt, rec, index">{{
                    index+1
                    }}</template>

                <template slot="sysdate">
                    {{sysdata}}
                </template>

                <template slot="name" slot-scope="txt, rec">
                    <a @click="handle_todashboard(rec)">{{txt}}</a>
                </template>

                <template slot="grade" slot-scope="txt">
                    <span v-if="txt=='01'">操作員</span>
                    <span v-if="txt=='08'">{{txt}}</span>
                </template>

                <template slot="salary_type" slot-scope="txt">
                    <span v-if="txt=='1'">正式工</span>
                    <span v-if="txt=='6'">臨時工</span>
                </template>
            </a-table>
        </a-modal>
    </div>
</template>

<script>

    import { reqPost } from '@/services/base'
    import { merge } from '@/utils/util'
    const sorter = (a, b, key) => (a[key] === b[key] ? 0 : a[key] > b[key] ? 1 : -1)

    const i18n = require('./i18n')
    export default {
        name: 'annuChart_list',
        props: {
            actionFix: String,
            vkey: { type: Number, default: 0 },
            title: { type: String, default: "List" },
            temptable: { type: Array, default: () => [] },
            deptid: { type: String, default: "" },
            dept: { type: String, default: "" },
        },
        i18n: merge(require('@/i18n'), i18n),
        data() {
            return {
                i18n,
                loadcount: 0,
                visible: false,
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
            sysdata() {
                return new Date().format('Y-m-d')
            },
            listCols() {
                return [
                    { key: 'no', align: 'center', width: 50, title: this.$t('no'), },
                    { key: 'name', width: 60, title: this.$t('name'), },
                    { key: 'empno', width: 90, title: this.$t('empno'), },
                    { key: 'annu_date', width: 90, title: this.$t('annu_date'), },
                    { key: 'sysdate', width: 90, title: this.$t('sysdate'), },
                    { key: 'annu', align: 'left', width: 90, title: this.$t('annu'), },
                    { key: 'salary_type', width: 90, title: this.$t('salary_type'), },
                    { key: 'grade', width: 90, title: this.$t('grade'), },
                ].map((e) => {
                    e.dataIndex = e.key.toLowerCase()
                    e.scopedSlots = { customRender: e.key }
                    e.align = e.align || 'left'
                    e.key !== 'no' ? (e.sorter = (a, b) => sorter(a, b, e.dataIndex)) : ''
                    return e
                })
            },
            loading() {
                return this.loadcount > 0
            },
        },
        watch: {
            vkey() {
                this.visible = true;

            },
        },
        created() {
        },
        mounted() { },
        methods: {
            handle_todashboard(e) {
                reqPost('WORKER', { prop: "getworkerid", empno: e.empno, deptid: this.deptid }).then(res => {
                    if (res.data != null) {
                        this.$router.push("../../../sys/worker/"+this.dept+"/dashboard?id=" + res.data+"&name="+e.name)
                        //window.open("../../../sys/worker/OQA/dashboard?id=" + res.data, "_self");
                    }
                    else {
                        this.$message.error("人員資本資料未維護")
                    }

                })

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