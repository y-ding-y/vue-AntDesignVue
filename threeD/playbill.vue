<template>
    <ef-main :isMainPage="true" :weather="true">
        <a-calendar v-model="calendar">
            <!-- :header-render="headerRender" -->
            <template slot="dateCellRender" class="events" slot-scope="value">
                <div v-for="(item,index) in tabledate" :key="index">
                    <div v-if="check(item.date_from,value)">
                        <b> {{item.coursename}}</b> <br>
                        {{item.date_from.slice(11,16)}}-{{item.date_to.slice(11,16)}}
                    </div>
                </div>
            </template>
        </a-calendar>
    </ef-main>

</template>

<script>
    import EfMain from '@/components/page/main'
    import { reqPost } from '@/services/base'
    import { moment } from '@/utils/util'
    import { mapState } from 'vuex'
    export default {
        props: {
            tabledate: { type: Array, default: () => [] },
        },
        data() {
            return {

                offdays: [],

                fm_month: "",
                fm_year: "",
                fm_day: "",
                to_month: "",
                to_year: "",
                to_day: "",
                calendar: null,
                values: {},
                loadcount: 0,
            }
        },
        created() {

        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
        },
        components: {
            EfMain
        },
        methods: {
            check(date, value) {
                return date.slice(0, 10) == value.format("yyyy-MM-DD");
            },

            handle_offduty(values) {
                var params = {
                    prop: "offduty",
                    empno: values.empno,
                    org_id: "476",
                    idate: values.workTime
                }; this.loadcount = 1;
                reqPost("WORKER_OQA", params).then(res => {
                    this.loadcount--;
                    this.offdays = res.data.map(e => {
                        e.date = e.date_fm.substr(0, 10)
                        return e;
                    })
                })
            },

            handle_calendar(values) {
                var params = {
                    prop: "calendar",
                    empno: values.empno,
                    org_id: "476",
                    idate: values.workTime
                };
                this.loadcount = 1;
                reqPost("WORKER_OQA", params).then(res => {
                    this.loadcount--;
                    this.tabledate = res.data.map(e => {
                        return {
                            type: "warning",
                            content: e.cdate,
                            date: e.cdate,
                            week: e.week,
                            iduty: e.iduty,
                            hh: e.itime_hh < 10 ? ('0' + e.itime_hh) : e.itime_hh,
                            mm: e.itime_mm < 10 ? ('0' + e.itime_mm) : e.itime_mm,
                            ss: e.itime_ss < 10 ? ('0' + e.itime_ss) : e.itime_ss,
                            kind: e.kind
                        }
                    })
                })
            },

            handle_select(value) {

                var temp = value.format('yyyy-MM-DD').split('-');
                var arr = [moment(new Date(this.fm_year, Number(temp[1]) - 1, 1), "yyyy-mm-dd"),
                moment(new Date(this.fm_year, Number(temp[1]), 0), "yyyy-mm-dd")]
                this.formItems[1].value = arr
                this.fm_month = temp[1]
                this.handle_search({ empno: this.values.empno, workTime: arr })
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