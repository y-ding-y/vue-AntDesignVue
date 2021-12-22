<template>
    <ef-main :isMainPage="true" :title="pagetitle" :loading="loading" @resize="resize" :dataTableId="tableId"
        :goback="true">

        <a-table :id="tableId" rowKey="id" :data-source="scoretable" :columns="scorecolumns" :size="sSize"
            :scroll="scroll" :bordered="true" :pagination="false">
            <template slot="no" slot-scope="txt, rec, index">
                <span>{{index+1}}</span>
            </template>

            <template slot="name" slot-scope="txt,rec">

                <a-popconfirm v-if="forms.prop=='searchallskill'">
                    <template slot="cancelText"> </template>
                    <a-icon slot="icon" type="''" />
                    <template slot="title">
                        <div style="width: 800px;">
                            <a-table :loading="loading1" class="basebg" size="small" :bordered="true" :columns="columns"
                                :pagination="false" :scroll="scrollskill" :data-source="skilltable" rowKey="id">
                                <template v-for="(item,index) in stationlist" :slot="item.id" slot-scope="txt">
                                    <div :key="index">
                                        <a-icon v-if="txt" type="check-circle" theme="twoTone"
                                            two-tone-color="#52c41a" />
                                        <a-icon v-else type="close-circle" theme="twoTone" two-tone-color="red" />
                                    </div>
                                </template>
                            </a-table>
                        </div>

                    </template>
                    <a @click="handle_toskill(rec)">{{txt}}</a>
                </a-popconfirm>

                <a v-if="forms.prop=='searchallscore'" @click=" handle_toscoremain(rec) ">{{txt}}</a>
                <a v-if="forms.prop=='searchallannu'" @click=" handle_toannu(rec) ">{{txt}}</a>
            </template>
            <template v-for="(item,index) in ['ondutydays','overtimehours','offdays','overtimedays']" :slot="item"
                slot-scope="txt">
                <span :key="index" v-if="txt>0">{{txt}}</span>
            </template>


            <template slot="on" slot-scope="txt,rec ">
                <a-tooltip>
                    <template slot="title">
                        <span v-html="rec.onstr"></span>
                    </template>
                    <a-tag color="green" v-if="Number(txt)>0"><b> {{txt}}</b></a-tag>
                </a-tooltip>
            </template>

            <template slot="off" slot-scope="txt,rec ">
                <a-tooltip>
                    <template slot="title">
                        <span v-html="rec.offstr"></span>
                    </template>
                    <a-tag color="red" v-if="Number(txt)>0"><b>{{txt}}</b></a-tag>
                </a-tooltip>
            </template>

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
        props: {
            dept: { type: String, default: "" },
        },
        i18n: merge(require('@/i18n'), i18n),
        data() {
            return {
                scoretable: [],
                loadcount: 0,
                loadcount1: 0,
                forms: {},
                temp_rec: {},
                allowClear: false,
                score_date: null,
                sSize: "middle",
                tableId: 'fpytable' + new Date().format('hiu'),
                workers: [],
                deptid: "",
                columntype: [],
                pagetitle: "",
                stationlist: [],
                skilltable: []
            };
        },
        watch: {
            temp_rec() {
                this.score_date = this.temp_rec.score_date;
            },
            dept(){ 
                this.$router.go(0) 
            },
        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { y: this.pageMinHeight - 200 }
            },
            scrollskill() {
                return { x: this.stationlist.length * 100 + 100, y: this.pageMinHeight - 200 }
            },
            loading() {
                return this.loadcount > 0;
            },
            loading1() {
                return this.loadcount1 > 0;
            },
            scorecolumns() {
                return [...[
                    { key: 'no', align: 'center', title: this.$t('no'), width: 90 },
                    { key: 'empno', align: 'left', title: this.$t('empno'), width: 100 },
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100 },
                ], ...this.columntype].map((e) => {
                    e.dataIndex = e.key
                    //    e.sorter = ['no'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e
                })
            },
            columns() {
                var arr = [
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100, fixed: 'left' },
                ];

                var temp = [];
                this.stationlist.map(e => {
                    temp.push({ key: e.id, title: e.name })
                })
                return [...arr, ...temp].map((e) => {
                    e.dataIndex = e.key
                    e.sorter = ['no', 'actions'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    //e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e
                })
            },
        },
        created() {
            this.loadcount = 1;
            reqPost('WORKER', { prop: "getdeptid", name: this.$props.dept }).then(res => {
                this.loadcount--;
                this.deptid = res.data;
                if (Object.values(this.$route.query).length > 0) {
                    this.forms.first = this.$route.query.first
                    this.forms.second = this.$route.query.second
                    this.forms.prop = this.$route.query.prop
                    this.forms.stationcount = this.$route.query.stationcount
                    switch (this.forms.prop) {
                        case "searchallscore":
                            this.pagetitle = "成績一覽表"
                            break;
                        case "searchallskill":
                            this.pagetitle = "技能一覽表"
                            break;
                        case "searchallannu":
                            this.pagetitle = "出勤一覽表"
                            break;
                    }
                    this.submit(this.forms)
                }
                else {
                    //
                }
            })
        },
        methods: {
            submit(values) {
                this.loadcount = 1;
                reqPost("WORKER", { prop: "getworker", id: this.deptid }).then(res => {
                    this.loadcount--;
                    var workers = [];
                    var empnos = [];
                    this.scoretable = [];
                    this.workers = res.data.map(e => {
                        workers.push(e.id)
                        empnos.push(e.empno);
                        return e
                    });
                    if (values.prop == "searchallannu") {
                        this.loadcount = 3;
                        var getovertimes = [];
                        var ondutydays = [];
                        var offdays = [];
                        reqPost("WORKER", { prop: "getovertimes", empno: empnos, org_id: "476", idate: [values.first, values.second] }).then(res => {
                            this.loadcount--;
                            getovertimes = res.data;
                            reqPost("WORKER", { prop: "ondutydays", empno: empnos, org_id: "476", idate: [values.first, values.second] }).then(res => {
                                this.loadcount--;
                                ondutydays = res.data.data;
                                reqPost("WORKER", { prop: "offdays", empno: empnos, org_id: "476", idate: [values.first, values.second] }).then(res => {
                                    this.loadcount--;
                                    offdays = res.data;
                                    this.handle_annu(getovertimes, ondutydays, offdays)
                                })
                            })
                        })



                    }
                    else {
                        this.loadcount = 1;
                        reqPost("WORKER", { prop: values.prop, id: workers, first: values.first, second: values.second ,deptid: this.deptid}).then(res => {
                            this.loadcount--;
                            //  this.scoretable = res.data;
                            switch (values.prop) {
                                case "searchallscore":
                                    this.hanlde_score(res.data)
                                    break;
                                case "searchallskill":
                                    this.handle_skill(res.data)
                                    break;
                            }

                        })
                    }

                })
            },
            hanlde_score(list) {
                this.columntype = [{ key: 'on', align: 'right', title: this.$t('on'), width: 120, sorter: (a, b) => Number(a.on) > Number(b.on) ? 1 : -1 },
                { key: 'off', align: 'right', title: this.$t('off'), width: 120, sorter: (a, b) => Number(a.off) > Number(b.off) ? 1 : -1 },];
                this.workers.map(e => {
                    e.on = 0;
                    e.onstr = "";
                    e.off = 0;
                    e.offstr = "";
                    list.map(a => {
                        if (a.worker == e.id && a.grade == "1") {
                            e.on++;
                            var temp = a.score_date + "：" + a.description
                            e.onstr = e.onstr + temp + "<br>"
                        }
                        else if (a.worker == e.id && a.grade == "-1") {
                            e.off++;
                            var temp2 = a.score_date + "：" + a.description
                            e.offstr = e.offstr + temp2 + "<br>"
                        }
                    })
                    this.scoretable.push(e)
                    //return e;
                })
            },
            handle_skill(list) {
                this.columntype = [{ key: 'pass', align: 'right', title: this.$t('pass'), width: 120, sorter: (a, b) => Number(a.pass) > Number(b.pass) ? 1 : -1 },
                { key: 'unpass', align: 'right', title: this.$t('unpass'), width: 120, sorter: (a, b) => Number(a.unpass) > Number(b.unpass) ? 1 : -1 },
                { key: 'total', align: 'right', title: this.$t('total'), width: 120 },];
                this.scoretable = list.map(e => {
                    e.id = e.worker
                    e.pass = e.count;
                    e.total = this.forms.stationcount
                    e.unpass = Number(this.forms.stationcount) - Number(e.count)
                    return e;
                });
            },
            handle_annu(getovertimes, ondutydays, offdays) {
                this.columntype = [
                    { key: 'ondutydays', align: 'right', title: this.$t('ondutydays'), width: 120, sorter: (a, b) => Number(a.ondutydays) > Number(b.ondutydays) ? 1 : -1 },
                    { key: 'overtimehours', align: 'right', title: this.$t('overtimehours'), width: 120, sorter: (a, b) => Number(a.overtimehours) > Number(b.overtimehours) ? 1 : -1 },
                    {
                        key: "top",
                        title: "請假次數",
                        children: [
                            { key: 'offdays', align: 'right', title: this.$t('offdays'), width: 120, sorter: (a, b) => Number(a.offdays) > Number(b.offdays) ? 1 : -1 },
                            { key: 'overtimedays', align: 'right', title: this.$t('overtimedays'), width: 120, sorter: (a, b) => Number(a.overtimedays) > Number(b.overtimedays) ? 1 : -1 },].map((e) => {
                                e.dataIndex = e.key
                                e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                                return e
                            })
                    },
                ];
                this.workers.map(e => {
                    getovertimes.some(a => {
                        if (e.empno == a.empno) {
                            e.overtimehours = a.count;
                            e.overtimedays = a.qty
                            return true
                        }
                        else {
                            e.overtimedays = 0; e.overtimehours = 0
                        }
                    })


                    ondutydays.some(a => {
                        if (e.empno == a.empno) {
                            e.ondutydays = a.count;
                            return true
                        }
                        else {
                            e.ondutydays = 0;
                        }
                    })

                    offdays.some(a => {
                        if (e.empno == a.empno) {
                            e.offdays = a.days;
                            return true
                        }
                        else {
                            e.offdays = 0;
                        }
                    })
                    this.scoretable.push(e)

                })
            },
            resize(size) {
                this.sSize = size
            },

            sorter(a, b, key) {
                const aI = a[key] ?? 1
                const bI = b[key] ?? 1
                return Math.sign(aI - bI)
            },

            handle_toscoremain(e) {
                this.$router.push("../../../sys/worker/OQA/scoremain?id=" + e.id + "&deptid=" + this.deptid + "&name=" + e.name)
                //window.open("../../../sys/worker/OQA/dashboard?id=" + e.id, "_self");
            },

            handle_toannu(e) {
                this.$router.push("/sys/worker/OQA/dutyinfo?empno=" + e.empno + "&first=" + this.forms.first + "&second=" + this.forms.second + "&name=" + e.name)
            },

            handle_toskill(e) {
                this.skilltable = [];
                this.loadcount1 = 1;
                reqPost('WORKER', { prop: "getskill", worker: e.worker }).then(data => {
                    this.loadcount--;
                    reqPost('WORKER', { prop: "getstation", worker: e.worker }).then(res => {
                        this.loadcount1--;
                        this.stationlist = res.data;

                        this.skilltable = [{ id: e.worker }].map(e => {
                            this.stationlist.map(a => {
                                e[a.id] = false;
                                data.data.some(b => {
                                    if (b.station == a.id) {
                                        e[a.id] = true
                                        e.name = b.name;
                                        this.name = b.name
                                        return true;
                                    }
                                })

                            })
                            return e;
                        });
                    })

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