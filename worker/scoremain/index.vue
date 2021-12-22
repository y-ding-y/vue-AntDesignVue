<template>
    <ef-main :isMainPage="true" :loading="loading" @resize="resize" :dataTableId="tableId" :goback="true">

        <a-table :id="tableId" rowKey="id" :data-source="scoretable" :columns="scorecolumns" :size="sSize"
            :bordered="true" :pagination="false" :scroll="scroll">
            <template slot="no" slot-scope="txt, rec, index">
                <span v-if="rec.grade=='1'" style="color: green;">{{index+1}}</span>
                <span v-else style="color: red;"> {{index+1}}</span>
            </template>

            <template v-for="(item,index) in ['name','description','score_date']" :slot="item" slot-scope="txt,rec">
                <span :key="index">
                    <span v-if="rec.grade=='1'" style="color: green;"> {{rec[item]}}</span>
                    <span v-else style="color: red;"> {{rec[item]}}</span>
                </span>
            </template>

            <template slot="grade" slot-scope="txt ">
                <span>
                    <span v-if="txt=='1'" style="color: green;"> 加點</span>
                    <span v-else style="color: red;">扣點</span>
                </span>
            </template>

            <!-- <template slot="description" slot-scope="txt,rec">
                <span v-if="rec.grade=='1'" style="color: green;"> {{rec.description}}</span>
                <span v-else style="color: red;"> {{rec.description}}</span>

            </template>

            <template slot="score_date" slot-scope="txt,rec">
                <span v-if="rec.grade=='1'" style="color: green;"> {{rec.score_date}}</span>
                <span v-else style="color: red;"> {{rec.score_date}}</span>

            </template> -->

        </a-table>


    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'
    import { merge, } from '@/utils/util'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    const i18n = require('./i18n')

    export default {
        name: 'workerScoremain',
        components: { EfMain },
        i18n: merge(require('@/i18n'), i18n),
        data() {
            return {
                scoretable: [],
                loadcount: 0,
                forms: {},
                temp_rec: {},
                allowClear: false,
                score_date: null,
                sSize: "middle",
                tableId: 'fpytable' + new Date().format('hiu'),
            };
        },
        watch: {
            temp_rec() {
                this.score_date = this.temp_rec.score_date;
            },
        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { y: this.pageMinHeight - 200 }
            },
            loading() {
                return this.loadcount > 0;
            },
            scorecolumns() {
                return [
                    { key: 'no', align: 'center', title: this.$t('no'), width: 90 },
                    { key: 'score_date', align: 'left', title: this.$t('score_date'), width: 180 },
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100 },
                    { key: 'description', align: 'left', title: this.$t('description'), },
                    { key: 'grade', align: 'left', title: this.$t('grade'), width: 120 },
                ].map((e) => {
                    e.dataIndex = e.key
                    //     e.sorter = ['no'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e
                })
            },
        },
        created() {
            if (Object.values(this.$route.query).length > 0) {
                this.forms.id = this.$route.query.id
                this.forms.first = this.$route.query.first
                this.forms.second = this.$route.query.second
                this.submit(this.forms)
            }
            else {
                //
            }
        },
        methods: {
            submit(values) {
                this.loadcount = 1;
                values.prop = "searchscore";
                reqPost("WORKER", values).then(res => {
                    this.loadcount--;
                    this.scoretable = res.data;
                })
            },
            resize(size) {
                this.sSize = size
            },
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
</style>