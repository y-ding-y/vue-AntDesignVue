<template>
    <ef-main :dataTableId="tableId" :isMainPage="true" :loading="loading" :form-items="formItems" @submit="submit">
        <a-table :id="tableId" class="basebg" :size="sSize" :bordered="true" :columns="columns" :data-source="tableData"
            :pagination="false" :indentSize="25" rowKey="id" :scroll="scroll">
            <template slot="no" slot-scope="txt, rec, i">
                {{ i + 1 }}
            </template>

            <template slot="empno" slot-scope="txt">
                <a-tooltip :title="txt">
                    {{empnoname}}
                </a-tooltip>
            </template>

            <template slot="status" slot-scope="txt, rec">
                <a-tag v-if="rec.record!=null" color="purple">已結束</a-tag>
                <a-tag v-else-if="rec.date_from>today" color="cyan">未開始</a-tag>
                <a-tag v-else-if="rec.date_from<=today && rec.date_to>today" color="orange">進行中</a-tag>
                <a-tag v-else-if="rec.date_to<today && rec.record==null" color="pink">待評分</a-tag>

            </template>

            <template slot="score" slot-scope="txt, rec">
                <a-tooltip>
                    <template slot="title">
                        <span v-if="rec.pass=='Y'">已通過</span>
                        <span v-if="rec.pass=='N'">未通過</span>
                    </template>
                    <a-tag v-if="rec.pass=='Y'" color="green">{{rec.score}}</a-tag>
                    <a-tag v-else-if="rec.pass=='N'" color="red">{{rec.score}}</a-tag>
                    <span v-else></span>
                </a-tooltip>
            </template>

            <template slot="daterange" slot-scope="txt, rec">
                <span>{{rec.date_from}}~{{rec.date_to}}</span>
            </template> 
        </a-table>
        <br>


    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'
    import EfMain from "@/components/page/main";
    import { merge } from '@/utils/util'
    import { mapState } from 'vuex'

    //厂别，投诉日期，投诉类别，投诉人，机种，问题描述，原因类别，上传文件
    const formItems = [
        { key: 'empno', type: 'input', },
    ]

    const i18n = require('./i18n')
    export default {
        name: 'trainingdept',
        components: { EfMain },
        i18n: merge(require('@/i18n'), i18n),
        props: {
            dept: { type: String, default: "" },
        },
        data() {
            return {
                tableId: 'fpytable' + new Date().format('hiu'),
                formItems,
                loadcount: 0,
                sSize: 'middle',
                tableData: [],
                values: {},
                empnoname: "",
            };
        },

        computed: {
            today() {
                return new Date().format('Y-m-d HH:mm:ss')
            },
            // empnoname() {
            //     return JSON.parse(localStorage.getItem(process.env.VUE_APP_USER_KEY)).name
            // },
            empno() {
                return JSON.parse(localStorage.getItem(process.env.VUE_APP_USER_KEY)).empno
            },
            scroll() {
                return { x: 10, y: this.pageMinHeight - 200 }
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),

            loading() {
                return this.loadcount > 0;
            },

            columns() {
                var arr = [
                    { key: 'no', width: 90, align: "center" },
                    { key: "empno", width: 100 },
                    { key: "coursename", width: 150, },
                    { key: "daterange", width: 300, },
                    { key: "score", width: 100, },
                    { key: "status", },
                    //,sorter: (a, b) => a.complaindate > b.complaindate ? 1 : -1
                ].map((e) => {
                    e.dataIndex = e.key
                    e.title = this.$t(e.key)
                    e.slots = { title: e.key + 'Title' }
                    e.scopedSlots = { customRender: e.slot ?? e.key }
                    return e
                })
                return arr;
            },
        },
        created() {
            this.handle_get();
            this.handle_searchempno(this.empno);
        },
        methods: {
            submit(values) {
                this.loadcount = 1;
                this.handle_searchempno(values.empno);
                reqPost('TRAINING', { prop: "getempno_class", empno: values.empno }).then(res => {
                    this.loadcount--;
                    this.tableData = res.data
                })
            },
            handle_get() {
                this.loadcount = 1;
                reqPost('TRAINING', { prop: "getempno_class", empno: this.empno }).then(res => {
                    this.loadcount--;
                    this.tableData = res.data
                })
            },

            handle_searchempno(e) {
                reqPost('ERP_HR_EMPLOYEE', {
                    empno: e,
                    select: "DISTINCT  name,empno"
                }).then(res => {
                    this.empnoname = res.data[0].name;
                })
            }
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

    .img {
        width: 2rem;
        height: 2rem;
        border-radius: .04rem;
        margin-left: .3rem;
        margin-top: .3rem;
        object-fit: cover;
    }
</style>