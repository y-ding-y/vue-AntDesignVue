<template>
    <ef-main :isMainPage="true" :loading="loading" :goback="true">

        <a-table rowKey="id" :data-source="scoretable" :columns="scorecolumns" :bordered="true" :pagination="false"
            :scroll="scroll">
            <template slot="no" slot-scope="txt, rec, index">
                <span> {{index+1}}</span>
            </template>

            <template slot="grade" slot-scope="txt,rec">
                <span v-if="!rec.isaction">
                    <a-tag v-if="txt=='1'" color="green">加點</a-tag>
                    <a-tag v-else-if="txt=='-1'" color="red">扣點</a-tag>
                </span>
                <a-radio-group v-if="rec.isaction" v-model="rec.grade">
                    <a-radio value="1">加點</a-radio>
                    <a-radio value="-1">扣點</a-radio>
                </a-radio-group>

            </template>

            <template slot="description" slot-scope="txt,rec">
                <div v-if="!rec.isaction">
                    <span v-if="rec.grade=='1'" style="color: green;"> {{rec.description}}</span>
                    <span v-else style="color: red;"> {{rec.description}}</span>
                </div>
                <textarea v-if="rec.isaction" v-model="rec.description"></textarea>
            </template>

            <template slot="score_date" slot-scope="txt,rec">
                <div v-if="!rec.isaction">
                    <span v-if="rec.grade=='1'" style="color: green;"> {{rec.score_date}}</span>
                    <span v-else style="color: red;"> {{rec.score_date}}</span>
                </div>
                <a-date-picker v-if="rec.isaction" @change="date_change" v-model="rec.score_date"
                    :allow-clear="allowClear" />
            </template>

            <template slot="actions" slot-scope="text, rec,index">
                <div v-if="!rec.isaction">
                    <a-button :class="'warning'" size="small" @click="handle_update(rec)">
                        <a-icon type="edit" />
                    </a-button>
                    &nbsp;
                    <a-popconfirm title="Are you sure delete ?" ok-text="Yes" cancel-text="No"
                        @confirm="handle_delete(rec,index)">
                        <a-button type="danger" size="small" ghost>
                            <a-icon type="delete" />
                        </a-button>
                    </a-popconfirm>
                </div>

                <div v-if="rec.isaction ">
                    <a-button :class="'success'" size="small"
                        @click="actionstatus=='add'?handle_saveadd(rec):handle_saveupdate(rec)">
                        <a-icon type="save" />
                    </a-button>
                    &nbsp;
                    <a-button size="small" type="danger" ghost @click="handle_close(rec,index)">
                        <a-icon type="close" />
                    </a-button>
                </div>
            </template>
        </a-table>
        <br>
        <a-button v-if="this.forms.worker" block type="primary" ghost @click="handle_add">Add</a-button>

    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'
    import { merge, moment } from '@/utils/util'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    const i18n = require('@/i18n')
    const formItems = [
        { key: 'worker', type: 'input', value: "2" },
    ]

    export default {
        name: 'workerScore',
        components: { EfMain },
        i18n: merge(require('./i18n'), i18n),
        props: {
            dept: { type: String, default: "" },
        },
        data() {
            return {
                formItems,
                scoretable: [],
                loadcount: 0,
                forms: { worker: "" },
                temp_rec: {},
                allowClear: false,
                score_date: null,
                actionstatus: "",
            };
        },
        watch: {
            temp_rec() {
                this.score_date = this.temp_rec.score_date;
            },
            dept() {
                this.$router.go(0)
            },
        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { x: 10, y: this.pageMinHeight - 200 }
            },
            loading() {
                return this.loadcount > 0;
            },
            scorecolumns() {
                return [
                    { key: 'no', align: 'center', title: this.$t('no'), width: 90 },
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100 },
                    { key: 'description', align: 'left', title: this.$t('description'), },
                    { key: 'grade', align: 'left', title: this.$t('score'), width: 120 },
                    { key: 'score_date', align: 'left', title: this.$t('score_date'), width: 180 },
                    { key: 'actions', align: 'left', title: this.$t('actions'), width: 120 },
                ].map((e) => {
                    e.dataIndex = e.key
                    e.sorter = ['no', 'actions'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e
                })
            },
            addforms() {
                return {
                    name: "",
                    worker: this.forms.worker,
                    dept: this.forms.deptid,
                    description: "",
                    grade: "1",
                    score_date: moment(new Date(), "YYYY-MM-DD"),
                    isaction: true,
                }
            }
        },
        created() {
            this.forms.deptname = this.$props.dept
            if (Object.values(this.$route.query).length > 0) {
                this.forms.deptid = this.$route.query.deptid
                this.forms.worker = this.$route.query.worker
                this.forms.deptname = this.$route.query.name
                this.addforms.name = this.$route.query.name;
                this.handle_search();
            }
            else {
                //
            }
        },
        methods: {
            handle_search() {
                this.loadcount = 1;
                reqPost("WORKER", { prop: "searchscore", id: this.forms.worker }).then(res => {
                    this.loadcount--;
                    this.scoretable = res.data.map(e => {
                        e.isaction = false;
                        return e;
                    });
                })
            },

            handle_update(rec) {
                this.actionstatus = "update"
                var flag = 1;
                for (var i of this.scoretable) {
                    if (i.isaction) {
                        this.$message.warning("有編輯尚未保存")
                        flag = -1;
                    }
                }
                if (flag == 1) {
                    this.temp_rec = { ...rec };
                    this.scoretable = this.scoretable.map(e => {
                        if (e.id == rec.id && flag == 1) {
                            e.score_date = moment(e.score_date, 'YYYY-MM-DD');
                            e.isaction = true;
                        }
                        return e;
                    })
                }

            },

            handle_delete(rec, index) {
                reqPost("WORKER", { prop: "deletescore", id: rec.id }).then(res => {
                    if (res.data > 0) {
                        this.i++;
                        this.$message.success("Delete Success")
                        this.scoretable.splice(index, 1);
                    }
                    else {
                        this.$message.error("Delete ERROR")
                    }
                })
            },

            date_change(date, dateString) {
                this.score_date = dateString || null;
            },

            handle_saveupdate(rec) {
                var params = { ...rec }
                params.score_date = this.score_date
                params.prop = "updatescore";
                reqPost("WORKER", params).then(res => {
                    if (res.data > 0) {
                        this.$message.success("Update Success")
                        this.handle_search();
                    }
                    else {
                        this.$message.error("Update Error")
                    }
                })
            },

            handle_close(rec, index) {
                if (this.actionstatus == "add") {
                    this.scoretable.splice(index, 1);
                }
                else if (this.actionstatus == "update") {
                    this.scoretable.splice(index, 1, this.temp_rec);
                }
            },

            handle_add() {
                const temp = { ...this.addforms }
                for (var i of this.scoretable) {
                    if (i.isaction) {
                        return this.$message.warning("请先保存当前！")
                    }
                }
                this.actionstatus = "add";
                temp.id = "id" + this.scoretable.length + 1;
                this.scoretable.push(temp)
            },
            handle_saveadd(e) {
                e.score = e.grade;
                e.prop = "insertscore"
                this.loadcount = 1;
                reqPost("WORKER", e).then(res => {
                    this.loadcount--;
                    if (res.data > 0) {
                        this.$message.success("Add Succcess")
                        this.handle_search();
                    }
                    else {
                        this.$message.error("Add Error")
                    }
                })

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