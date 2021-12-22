<template>
    <ef-main :isMainPage="true" :loading="loading"  :goback="true">
        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="column" :pagination="false"
            :data-source="tableData" rowKey="id" :scroll="scroll">
            <template slot="no" slot-scope="txt, rec, index">
                {{index+1}}
            </template>

            <template slot="name">
                {{values.name}}
            </template>

            <template slot="cdate" slot-scope="txt, rec">

                <span v-if="rec.kind=='P'">
                    {{txt}}
                </span>
                <b v-else>
                    <span v-if="rec.remarks==null">{{txt}}</span>
                    <a-tooltip v-else :title="rec.remarks">
                        <span>{{txt}}<span style="color: red;">*</span></span>
                    </a-tooltip>
                </b>
            </template>

            <template slot="offduty" slot-scope="txt,rec">
                <div v-if="rec.cdate <= today">
                    <span v-if="rec.kind=='P' && !rec.offduty && rec['+']">
                        <a-tag color="green">正常出勤</a-tag>
                    </span>

                    <span v-if="rec.kind=='P' && !rec.offduty && rec['+']==null">
                        <a-tag color="purple">考勤資料缺失</a-tag>
                    </span>


                    <span v-if="rec.kind!='P' && !rec.offduty">
                        <a-tag v-if="rec['+']" color="cyan">加班</a-tag>
                        <a-tag v-else color="orange">休息</a-tag>
                    </span>

                    <span v-if="rec.kind=='P' && rec.offduty  ">
                        <a-tag v-if="['O','Y'].includes(rec.absent_code)" color="green">{{rec.reason}}</a-tag>

                        <a-tooltip v-else :title="rec.reason">
                            <a-tag color="red">請假</a-tag>
                        </a-tooltip>

                    </span>

                    <!-- <span v-if="rec.kind=='P' && rec.offduty && rec.absent_code=='Y'">
                        <a-tag color="red">{{rec.reason}}</a-tag>
                    </span> -->
                </div>

            </template>

            <template slot="notes" slot-scope="txt, rec">
                <div v-if="rec.cdate <= today && rec['+']">
                    <span v-if="rec.overtime!=null &&rec.overhour!='0'">
                        加班時間{{rec.overtime}}&nbsp;&nbsp;&nbsp;原因：{{rec.notes_detail}}
                    </span>

                    <span v-if="rec.overhour==null && rec.overtime==null && rec.hours">
                        <a-tag color="cyan">已報加班，但暫無實際加班信息</a-tag>  
                    </span>
                </div>

            </template>

            <template slot="overhour" slot-scope="txt,rec ">
                <span v-if="txt!='0' && rec.cdate <= today && txt!=null">
                    {{Number(txt)}}
                </span>
            </template>

        </a-table>
    </ef-main>

</template>
<script>
    import EfMain from '@/components/page/main'
    import { merge } from '@/utils/util'
    import { reqPost } from '@/services/base'
    import { mapState } from 'vuex'

    const i18n = require('./i18n')

    export default {
        name:"dutyInfo",
        components: { EfMain, },
        i18n: merge(require('@/i18n'), i18n),
        data() {
            return {
                tabledate: [],
                offdays: [],
                values: {},
                loadcount: 0,
                tableData: [],
                sSize: "middle",
                tableId: 'oqaworkerdstation' + new Date().format('hiu'),
            }
        },
        created() {
            if (Object.values(this.$route.query).length > 0) {
                this.values.empno = this.$route.query.empno
                this.values.first = this.$route.query.first
                this.values.second = this.$route.query.second
                this.values.name = this.$route.query.name
                this.handle_search(this.values)
            }
            else {
                //
            }

        },
        computed: {
            today() {
                return new Date().format('Y-m-d')
            },
            scroll() {
                return { x: 1300, y: this.pageMinHeight - 130 }
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),
            loading() {
                return this.loadcount > 0;
            },
            column() {
                var arr = [
                    { key: 'no', align: 'center', title: this.$t('no'), width: 60, fixed: "left" },
                    { key: 'name', align: 'center', title: this.$t('name'), width: 100, fixed: "left" },
                    { key: 'cdate', align: 'left', title: this.$t('date'), width: 130, fixed: "left" },
                    { key: 'week', align: 'left', title: this.$t('week'), width: 60 },
                    { key: '+', align: 'left', title: this.$t('onduty'), width: 100 },
                    { key: '-', align: 'left', title: this.$t('offduty'), width: 100 },
                    { key: 'offduty', align: 'left', title: this.$t('status'), width: 150 },
                    { key: 'overhour', align: 'left', title: this.$t('overhour'), width: 120 },
                    { key: 'notes', align: 'left', title: this.$t('notes'), },
                    //{ key: 'type', align: 'left', title: this.$t('type'), width: 100 },
                    //{ key: 'actions', align: 'left', title: this.$t('actions'), width: 100 },
                    //{ key: 'qty', align: 'right', title: this.$t("qty") },
                ].map((e) => {
                    e.dataIndex = e.key
                  //  e.sorter = ['no'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e;
                })
                return arr;
            },
        },
        methods: {
            handle_search(values) {
                this.handle_offduty(values);
            },

            handle_offduty(values) {
                var params = {
                    prop: "offduty",
                    empno: values.empno,
                    org_id: "476",
                    idate: [values.first, values.second]
                }; this.loadcount = 1;
                this.offdays = [];
                reqPost("WORKER", params).then(res => {
                    this.loadcount--;
                    res.data.map(e => {
                        if (e.days == "1") {
                            e.date = e.date_fm.substr(0, 10)
                            this.offdays.push({ ...e })
                        }
                        else {
                            var cur_date = e.date_fm.substr(0, 10);
                            for (var i = 0; i < Number(e.days); i++) { 
                                new Date(cur_date).setDate(new Date(cur_date).getDate() + 1)
                                var date = new Date(cur_date);
                                date.setDate(date.getDate() + i); 
                                e.date = date.format('Y-m-d')
                                this.offdays.push({ ...e })
                            }

                            // console.log(date.getFullYear() + "-" + (date.getMonth() + 1) + "-" + date.getDate());
                        }
                    })
                    this.handle_calendar(values);
                })
            },
            handle_calendar(values) {
                var params = {
                    prop: "calendar",
                    empno: values.empno,
                    org_id: "476",
                    idate: [values.first, values.second]
                };
                this.loadcount = 1;
                reqPost("WORKER", params).then(res => {
                    this.loadcount--;
                    this.tableData = res.data
                    var temp = this.dedupe(res.data.map(e => {
                        return e.cdate
                    }))

                    this.tableData = temp.map((e, index) => {
                        var arr = {};
                        // if (e.cdate == o.date_fm.substr(0, 10)) {
                        //     arr.offduty = true;
                        // }
                        res.data.some(a => {
                            if (e == a.cdate) {
                                arr.id = index;
                                arr[a.iduty] = (a.itime_hh < 10 ? ('0' + a.itime_hh) : a.itime_hh) + ":"
                                    + (a.itime_mm < 10 ? ('0' + a.itime_mm) : a.itime_mm) + ":"
                                    + (a.itime_ss < 10 ? ('0' + a.itime_ss) : a.itime_ss);
                                arr.cdate = a.cdate;
                                arr.kind = a.kind;
                                arr.week = a.week;
                                arr.empno = a.empno;
                                arr.remarks = a.remarks;
                                arr.notes_detail = a.notes_detail;
                                a.act_hh_fm && (arr.overtime = (a.act_hh_fm < 10 ? ('0' + a.act_hh_fm) : a.act_hh_fm) + ":"
                                    + (a.act_mm_fm < 10 ? ('0' + a.act_mm_fm) : a.act_mm_fm) + "~"
                                    + (a.act_hh_to < 10 ? ('0' + a.act_hh_to) : a.act_hh_to) + ":"
                                    + (a.act_mm_to < 10 ? ('0' + a.act_mm_to) : a.act_mm_to));
                                arr.overhour = a.act_hours;
                                arr.hours=a.hours;
                            }
                        })
                        return arr
                    }).map(e => {
                        this.offdays.some(a => {
                            if (e.cdate == a.date.substr(0, 10)) {
                                e.offduty = true;
                                e.absent_code = a.absent_code;
                                e.reason = a.reason
                                return true

                            }
                            else {
                                e.offduty = false;
                            }
                        })
                        return e;
                    })
                    // this.tabledate = res.data.map(e => {
                    //     return {
                    //         type: "warning",
                    //         content: e.cdate,
                    //         date: e.cdate,
                    //         week: e.week,
                    //         iduty: e.iduty,
                    //         hh: e.itime_hh < 10 ? ('0' + e.itime_hh) : e.itime_hh,
                    //         mm: e.itime_mm < 10 ? ('0' + e.itime_mm) : e.itime_mm,
                    //         ss: e.itime_ss < 10 ? ('0' + e.itime_ss) : e.itime_ss,
                    //         kind: e.kind
                    //     }
                    // })
                })
            },

            dedupe(array) {
                return Array.from(new Set(array));
            },
        },
    };
</script>
<style scoped>
    .events {
        list-style: none;
        margin: 0;
        padding: 0;
    }

    .events .ant-badge-status {
        overflow: hidden;
        white-space: nowrap;
        width: 100%;
        text-overflow: ellipsis;
        font-size: 12px;
    }

    .notes-month {
        text-align: center;
        font-size: 28px;
    }

    .notes-month section {
        font-size: 28px;
    }
</style>