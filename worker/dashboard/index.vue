<template>
    <ef-main :isMainPage="true" :form-items="formItems" @submit="submit" :openSubmit="true">
        <a-descriptions layout="vertical" bordered>
            <a-descriptions-item>
                <template slot="label">
                    <a-button @click="handle_skill" type="primary" ghost>技能</a-button>
                </template>
                <a-descriptions bordered size="small">
                    <a-descriptions-item :span="3">
                        <template slot="label"> <b>Total崗位數</b> </template>
                        {{totalstation}}
                    </a-descriptions-item>
                    <a-descriptions-item :span="3">
                        <template slot="label"> <b>考核Pass</b> </template>
                        {{passstation}}
                    </a-descriptions-item>
                    <a-descriptions-item :span="3">
                        <template slot="label"> <b>未考核數</b> </template>
                        {{failstation}}
                    </a-descriptions-item>
                </a-descriptions>
            </a-descriptions-item>
            <a-descriptions-item>
                <template slot="label">
                    <a-button @click="handle_erp" type="primary" ghost>出勤</a-button>
                </template>
                <a-descriptions bordered size="small">
                    <a-descriptions-item :span="3">
                        <template slot="label"> <b>工作天數</b> </template>
                        test1
                    </a-descriptions-item>
                    <a-descriptions-item :span="3">
                        <template slot="label"> <b>加班小時</b> </template>
                        test3
                    </a-descriptions-item>
                    <a-descriptions-item :span="3">
                        <template slot="label"> <b>請假次數</b> </template>
                        <a-descriptions bordered size="small" layout="vertical">
                            <a-descriptions-item>
                                <template slot="label"> <b>工作天</b> </template>
                                test1
                            </a-descriptions-item>
                            <a-descriptions-item>
                                <template slot="label"> <b>加班</b> </template>
                                test3
                            </a-descriptions-item>
                        </a-descriptions>
                    </a-descriptions-item>
                </a-descriptions>
            </a-descriptions-item>
            <a-descriptions-item>
                <template slot="label">
                    <a-button @click="handle_score" type="primary" ghost>成績單</a-button>
                </template>
                <a-descriptions bordered size="small">
                    <a-descriptions-item :span="3" v-for="(item,index) in scorelist" :key="index">
                        <template slot="label">
                            <b v-if="item.grade=='1'">加點</b>
                            <b v-else-if="item.grade=='-1'">扣點</b>
                        </template>
                        <a-tag v-if="item.grade=='1'" color="green"> {{item.count}} </a-tag>
                        <a-tag v-if="item.grade=='-1'" color="red"> {{item.count}} </a-tag>
                    </a-descriptions-item>
                </a-descriptions>
            </a-descriptions-item>
        </a-descriptions>

        <br>
       
        <a-card title="成績單" size="small">
           
            <a-table rowKey="id" :data-source="scoretable" :columns="scorecolumns" :bordered="true" :pagination="false"
                size="small">
                <template slot="no" slot-scope="txt, rec, index">
                    <span> {{index+1}}</span>
                </template>
                <template slot="grade" slot-scope="txt">
                    <a-tag v-if="txt=='1'" color="green">加點</a-tag>
                    <a-tag v-else-if="txt=='-1'" color="red">扣點</a-tag>
                </template>
            </a-table>
        </a-card>

    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'
    import { merge } from '@/utils/util'
    import EfMain from '@/components/page/main'

    const i18n = require('./i18n')
    const formItems = [
        { key: 'worker', type: 'input', value: "2" },
    ]

    export default {
        name: 'oqaworkerdscore',
        components: { EfMain },
        i18n: merge(require('./i18n'), i18n),
        data() { 
            return {
                formItems,
                scoretable: [],
                scorelist: [],

                totalstation: 0,
                passstation: 0,

                forms: {worker:""},
 
            };
        },
        watch: {

        },
        computed: {
            failstation() {
                return Number(this.totalstation) - Number(this.passstation)
            },
            scorecolumns() {
                return [
                    { key: 'no', align: 'center', title: this.$t('no'), width: 90 },
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100 },
                    { key: 'description', align: 'left', title: this.$t('description'), },
                    { key: 'grade', align: 'left', title: this.$t('grade'), width: 90 },
                    { key: 'score_date', align: 'left', title: this.$t('score_date'), width: 180 },
                ].map((e) => {
                    e.dataIndex = e.key
                    e.sorter = ['no', 'actions'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e
                })
            },
        },
        created() {

        },
        methods: {
            

            submit(values) {
                this.forms = { ...values }
                reqPost("WORKER_OQA", { prop: "searchscorecount", id: values.worker }).then(res => {
                    this.scorelist = res.data;
                })

                reqPost("WORKER_OQA", { prop: "stationcount", id: values.worker }).then(res => {
                    this.totalstation = res.data[0].count;
                })

                reqPost("WORKER_OQA", { prop: "skillcount", id: values.worker }).then(res => {
                    this.passstation = res.data[0].count;
                })
            },

            handle_score() { 

                reqPost("WORKER_OQA", { prop: "searchscore", id: this.forms.worker }).then(res => {
                    this.scoretable = res.data;
                })
            },

            handle_erp() {
                console.log("123")
            },

            handle_skill() {
                console.log("123")
            },
        },
    };
</script>