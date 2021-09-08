<template>
    <ef-main :isMainPage="true" :form-items="formItems" @submit="submit" :openSubmit="true">

        <a-table rowKey="id" :data-source="scoretable" :columns="scorecolumns" :bordered="true" :pagination="false">
            <template slot="no" slot-scope="txt, rec, index">
                <span> {{index+1}}</span>
            </template>
            <template slot="grade" slot-scope="txt">
                <a-tag v-if="txt=='1'" color="green">加點</a-tag>
                <a-tag v-else-if="txt=='-1'" color="red">扣點</a-tag>
            </template>
        </a-table>


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

                forms: { worker: "" },

            };
        },
        watch: {

        },
        computed: {
            scorecolumns() {
                return [
                    { key: 'no', align: 'center', title: this.$t('no'), width: 90 },
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100 },
                    { key: 'description', align: 'left', title: this.$t('description'), },
                    { key: 'grade', align: 'left', title: this.$t('grade'), width: 120 },
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
                reqPost("WORKER_OQA", { prop: "searchscore", id: this.forms.worker }).then(res => {
                    this.scoretable = res.data;
                })
            },

            handle_score() {

            },


        },
    };
</script>