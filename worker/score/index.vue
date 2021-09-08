<template>
    <ef-main :isMainPage="true" :loading="loading" :form-items="formItems" @submit="submit">
        <upload-form :key="i" :isMainPage="false" :istitile="false" :form-items="scoreitems" @submit="handle_score">
        </upload-form>
    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'

    import { merge } from '@/utils/util'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    import UploadForm from '@/components/page/uploadform'
    const formItems = [
        { key: 'dept', type: 'select', paraKey: "WORKER.dept", allowClear: false, },
    ]

    const i18n = require('./i18n')
    export default {
        name: 'oqaworkerdscore',
        components: { EfMain, UploadForm },
        i18n: merge(require('./i18n'), i18n),
        data() {
            return {
                tableId: 'oqaworkerdstation' + new Date().format('hiu'),
                loadcount: 0,
                formItems,
                scoreitems: [],
                deptlist: [],
                deptid: "",
                deptname: "",
                workerlist: [],
                i: 0
            };
        },
        watch: {

        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { y: this.pageMinHeight - 200 }
            },
            loading() {
                return this.loadcount > 0;
            },

            forms() {
                return {
                    name: "",
                    dept_name: this.deptname,
                    dept: "",
                    salary_type: "",
                    annu_date: "",
                    lev_date: "",
                    station: "",
                }
            }
        },
        created() {
            this.loadcount = 1;
            reqPost('PARA_SET', { paras: [{ "key": "WORKER.dept" }] }).then(res => {
                this.loadcount--;
                if (res.data[0].data.length > 0) {
                    this.deptlist = res.data;
                    this.formItems[0].value = res.data[0].data[0].key;
                    this.deptid = res.data[0].data[0].key;
                    this.deptname = res.data[0].data[0].name;
                    this.handle_searchworker();
                }
                else {
                    this.$message.error("請先維護部門")
                }
            })

        },
        methods: {
            handle_score(values) {
                console.log(values);
                values.prop = "insertscore"
                this.loadcount = 1;
                reqPost("WORKER_OQA", values).then(res => {
                    this.loadcount--;
                    if (res.data > 0) {
                        this.$message.success("Add Succcess")
                    }
                    else {
                        this.$message.error("Add Error")
                    }
                })
            },


            submit(values) {
                this.deptid = values.dept;
                this.handle_searchworker();
            },

            handle_searchworker() {
                this.loadcount = 1;
                var params = {
                    prop: "getworker",
                    id: this.deptid
                }
                reqPost('WORKER_OQA', params).then(res => {
                    this.loadcount--;
                    this.workerlist = res.data;
                    var temp = res.data.map(e => {
                        return {
                            key: e.id,
                            title: e.name
                        }
                    })
                    this.i++;
                    console.log(temp)
                    this.scoreitems = [
                        { key: 'worker', type: 'select', options: temp, allowClear: false, span: "1", style: "width:100%" },
                        {
                            key: 'score_date', type: 'date', expandDayTime: true, format: 'YYYY-MM-DD',
                            maxDays: 366, value: new Date().format('Y-m-d'), allowClear: false, span: "2"
                        },
                        { key: 'description', type: 'textarea', allowClear: false, span: "3" },
                        { key: 'score', type: 'radio', options: [{ key: "1", value: "1", title: "加點", }, { key: "-1", value: "-1", title: "扣點" }], value: "1", span: "3", },
                    ];
                })
            },


        },
    };
</script>