<template>
    <ef-main :isMainPage="true" :form-items="formItems" @submit="submit" :loading="loading" >
        <a-calendar @panelChange="handle_select" :header-render="headerRender" v-model="calendar">
            <!-- :header-render="headerRender" -->
            <template slot="dateCellRender" class="events" slot-scope="value">
                <div v-for="(item,index) in tabledate" :key="index">
                    <div v-if="check(item.date,value)">
                        <div v-if="item.kind!='P'" style="color: rgb(82, 136, 75);">
                            <span v-if="item.iduty=='+'">上班：{{item.hh+':'+item.mm+':'+item.ss}}</span>
                            <span v-if="item.iduty=='-'">下班：{{item.hh+':'+item.mm+':'+item.ss}}</span>
                        </div>
                        <div v-else>
                            <span v-if="item.iduty=='+'">上班：{{item.hh+':'+item.mm+':'+item.ss}}</span>
                            <span v-if="item.iduty=='-'">下班：{{item.hh+':'+item.mm+':'+item.ss}}</span>
                        </div>
                    </div>
                    <span></span>
                </div>

                <div v-for="(item,index) in offdays" :key="'off'+index">
                    <div v-if="check(item.date,value)">
                        <a-tag color="red"><b>缺勤</b></a-tag>
                        <br>
                        時間：{{item.time_hh_fm}}:{{item.time_mm_fm}}-{{item.time_hh_to}}:{{item.time_mm_to}}

                    </div>
                    <span></span>
                </div>
            </template>
            <!-- <ul slot="dateCellRender" slot-scope="value" class="events">
                <li v-for="(item,index) in tabledate" :key="index">
                    <a-badge :status="item.type" :text="index" />
                </li>
            </ul> -->
        </a-calendar>
    </ef-main>

</template>
<script>
    import EfMain from '@/components/page/main'
    import { reqPost } from '@/services/base'
    import { moment } from '@/utils/util'
    const formItems = [
        { key: 'empno', type: 'input', value: "EP769" },
        {
            key: 'date',
            dataIndex: 'workTime',
            type: 'range',
            expandDayTime: true,
            format: 'YYYY-MM-DD',
            maxDays: 366,
            value: 'week',
            allowClear: false,
        },
    ]

    export default {
        components: { EfMain, },
        data() {
            return {
                tabledate: [],
                offdays: [],
                formItems,
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
            if (this.$route.query.empno) {
                this.formItems[0].value = this.$route.query.empno;
                console.log( this.formItems[1].value)
            }
            else {
                this.formItems[0].value = "";
            }
        },
        computed: {
            loading() {
                return this.loadcount > 0;
            },
        },
        methods: {
            submit(values) {
                this.values = { ...values }
                console.log(values)
                var a = values.workTime[0].substr(0, 10).split('-')
                this.fm_year = a[0];
                this.fm_month = a[1];
                this.fm_day = a[2]
                var b = values.workTime[1].substr(0, 10).split('-')
                this.to_year = b[0];
                this.to_month = b[1];
                this.to_day = b[2]
                this.calendar = moment(this.fm_year + "-" + this.fm_month, "YYYY-MM-DD");
                this.handle_search(this.values);
            },
            check(date, value) {
                return date == value.format("yyyy-MM-DD");
            },

            handle_search(values) {
                this.handle_offduty(values);
                this.handle_calendar(values);
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
                    console.log(this.offdays)
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
                console.log(value.format('yyyy-MM-DD'))
                var temp = value.format('yyyy-MM-DD').split('-');
                var arr = [moment(new Date(this.fm_year, Number(temp[1]) - 1, 1), "yyyy-mm-dd"),
                moment(new Date(this.fm_year, Number(temp[1]), 0), "yyyy-mm-dd")]
                this.formItems[1].value = arr
                this.fm_month = temp[1]
                this.handle_search({ empno: this.values.empno, workTime: arr })
            },

            headerRender() {
                var calendar = this.fm_month
                return (
                    <div style={{ textAlign: "center", fontWeight: "500", fontSize: "30px" }}>
                        {`${calendar}`}  月出勤記錄
                    </div>
                );
                // const start = 0;
                // const end = 12;
                // const monthOptions = [];

                // const current = value.clone();
                // const localeData = value.localeData();
                // const months = [];
                // for (let i = 0; i < 12; i++) {
                //     current.month(i);
                //     months.push(localeData.monthsShort(current));
                // }

                // for (let index = start; index < end; index++) {
                //     monthOptions.push(
                //         <a-select-option class="month-item" key={`${index}`}>
                //             {months[index]}
                //         </a-select-option>,
                //     );
                // }
                // const month = value.month();

                // const year = value.year();
                // const options = [];
                // for (let i = year - 10; i < year + 10; i += 1) {
                //     options.push(
                //         <a-select-option key={i} value={i} class="year-item">
                //             {i}
                //         </a-select-option>,
                //     );
                // }
                // return (
                //     <div style={{ display: "flex", justifyContent: "flex-end",  }}>
                //         <div> <a-select
                //             dropdownMatchSelectWidth={false}
                //             class="my-year-select"
                //             onChange={newYear => {
                //                 const now = value.clone().year(newYear);
                //                 onChange(now);
                //             }}
                //             value={String(year)}
                //         >
                //             {options}
                //         </a-select></div>

                //         <div>   <a-select
                //             dropdownMatchSelectWidth={false}
                //             value={String(month)}
                //             onChange={selectedMonth => {
                //                 const newValue = value.clone();
                //                 newValue.month(parseInt(selectedMonth, 10));
                //                 onChange(newValue);
                //             }}
                //         >
                //             {monthOptions}
                //         </a-select></div>


                //     </div>
                // );
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