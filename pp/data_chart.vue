<template>
    <ef-main :id="imageId" :isMainPage="true" title="" :loading="loading" :openSubmit="true" :form-items="formItems"
        :imageDomId="imageId" @submit="submit" @change="handle_change">

        <a-radio-group v-model="chart_type" >
            <a-radio-button value="record">
                {{$t('record') }}
            </a-radio-button>
            <a-radio-button value="added">
               {{ $t('added') }}
            </a-radio-button>
        </a-radio-group>
<br>
        <ef-chart class="basebg" :padding="[24, 64, 48, 64]" :scale="scale" :series="series" :data="chart_data"
            :height="height">
        </ef-chart>
    </ef-main>
</template>

<script>

    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    import { merge, } from '@/utils/util'
    import EfChart from '@/components/chart/Chart'

    const formItems = [{
        key: 'date',
        dataIndex: 'iTime',
        type: 'range',
        expandDayTime: true,
        format: 'YYYY-MM-DD',
        maxDays: 30,
        value: 'today',
        allowClear: false,
    },
    {
        key: 'type',
        type: 'select',
        paraKey: 'SfcsPp.TYPE',
        allowClear: false,
        value: "101"
    },
    {
        key: 'equip',
        type: 'select',
        value: "9168",
        allowClear: false,
        options: [],
    },
    {
        key: 'seq',
        type: 'select',
        value: "1",
        mode: 'multiple',
        allowClear: false,
        options: [],
    },]

    export default {
        name: 'data_chart',
        i18n: merge(require('@/i18n'), require('./i18n')),
        components: { EfMain, EfChart },
        data() {
            return {
                imageId: 'fpyimage' + new Date().format('hiu'),
                loadcount: 0,
                formItems,
                chart_data: [],
                chartHeight: "small",
                forms: { type: "101", },
                data: 0,
                tablist: [],
                activedsn: 0,
                chart_type: "record",
            }
        },
        computed: {
            recordlist() {
                var temp_hour = []
                var res = []
                var days = this.GetNumberOfDays(this.forms.iTime[0], this.forms.iTime[1]);

                for (var i = 0; i < 24; i++) {
                    if (i < 10) {
                        temp_hour.push("0" + i + ":00")
                        temp_hour.push("0" + i + ":30")
                    }
                    else {
                        temp_hour.push(i + ":00")
                        temp_hour.push(i + ":30")
                    }
                }

                var nowDate = new Date(this.forms.iTime[0]);
                var year = nowDate.getFullYear();
                var month = nowDate.getMonth() + 1;
                var date = nowDate.getDate();

                for (var m = 0; m < days; m++) {
                    if (year % 4 == 0 && year % 100 != 0 || year % 400 == 0) {//console.log("闰年"); 
                        if (month == 2) {
                            if (date > 29) {
                                date = 1; month++;
                                if (month > 12) { year++ }
                            }
                        }
                        else {
                            if ([1, 3, 5, 7, 8, 10, 12].includes(month)) {
                                if (date > 31) {
                                    date = 1; month++;
                                    if (month > 12) { year++ }
                                }
                            }
                            else {
                                if (date > 30) {
                                    date = 1; month++;
                                    if (month > 12) { year++ }
                                }
                            }
                        }
                    } else {
                        if (month == 2) {
                            if (date > 28) {
                                date = 1; month++;
                                if (month > 12) { year++ }
                            }
                        }
                        else {
                            if ([1, 3, 5, 7, 8, 10, 12].includes(month)) {
                                if (date > 31) {
                                    date = 1; month++;
                                    if (month > 12) { year++ }
                                }
                            }
                            else {
                                if (date > 30) {
                                    date = 1; month++;
                                    if (month > 12) { year++ }
                                }
                            }
                        }
                    }
                    temp_hour.map(e => {
                        res.push(year + "-" + (month < 10 ? ("0" + month) : month) + "-" + (date < 10 ? ("0" + date) : date) + " " + e)
                    })
                    date++;
                }
                return res;
            },

            ...mapState('setting', ['lang', 'pageMinHeight']),
            loading() {
                return this.loadcount > 0
            },
            scale() {
                return [
                    {
                        dataKey: 'name',
                        type: 'cat',
                        tickCount: 12,
                        // range: [0, 1]
                    },
                    // {
                    //     dataKey: 'qty',
                    //     tickCount: 10,
                    // },
                    {
                        dataKey: 'record',
                        tickCount: 9,
                    },
                    {
                        dataKey: 'added',
                        tickCount: 5,
                    },
                ]
            },

            series() {
                return [
                    {
                        quickType: 'line',
                        position: this.chart_type == "record" ? "name*record" : 'name*added',
                        color: 'seqname',
                    },


                ]
            },

            height() {
                //  return this.pageMinHeight - 100
                return 400
            },


        },
        watch: {
            forms(newval, oldval) {
                if (oldval.type != newval.type) {
                    this.handle_equips();
                    this.handle_cols();
                }
            }
        },
        created() {
            this.handle_equips();
            this.handle_cols();
        },
        mounted() { },
        methods: {
            GetNumberOfDays(date1, date2) {//获得天数  
                var a1 = Date.parse(new Date(date1));
                var a2 = Date.parse(new Date(date2));
                var day = parseInt((a2 - a1) / (1000 * 60 * 60 * 24)) + 1;//核心：时间戳相减，然后除以天数
                return day
            },

            submit(values) {
                this.forms = { ...values }


                this.handle_data(values);
            },



            handle_change(values) {
                this.forms = { ...values };
            },
            handle_equips() {
                this.loadcount = 1;
                reqPost("SFCS_PP", { prop: "consts", key: "GTKEQUIP(" + this.forms.type + ")" })
                    .then(res => {
                        this.loadcount--
                        var temp = res.data.map(e => {
                            return {
                                key: e.key,
                                title: e.name,
                            }
                        })
                        this.formItems[2].options = temp;
                        this.formItems[2].value = temp[0].key || "";
                    })
            },

            handle_cols() {
                this.loadcount = 1;
                reqPost("SFCS_PP", {
                    prop: "attr", order: "seq", orgId: "476",
                    equip: this.forms.type == "101" ? "9168" : "2117", type: this.forms.type,
                    select: "DISTINCT seq as SEQ,unit,dataType AS DATATYPE"
                }).
                    then(res => {
                        this.loadcount--;
                        var temp = res.data.map(e => {
                            return {
                                key: e.seq,
                                title: e.datatype,
                            }
                        })
                        this.formItems[3].options = temp;
                        this.formItems[3].value = temp[0].key || "";
                    })
            },

            handle_data(values) {
                var params = {
                    prop: "data",
                    type: values.type,
                    orgId: "476",
                    date: values.iTime,
                    equip: values.equip,
                    seq: values.seq,
                    order: "creationDate DESC",
                    select: "record as RECORD,unit AS UNIT,seq AS SEQ,equip AS EQUIP," +
                        "creationDate as CREATIONDATE,dataType as DATATYPE,added as ADDED",
                    limit: "3000"
                };
                this.loadcount = 1;
                reqPost("SFCS_PP", params)
                    .then(res => {
                        this.loadcount--;
                        var result = [];
                        //    var name = ""; 
                        if (typeof (values.seq) == "string") {
                            for (var i = 1; i <= this.recordlist.length; i++) {
                                var temp = {};
                                temp.name = this.recordlist[i - 1]
                                temp.seq = values.seq;
                                res.data.map(e => {
                                    if (e.creationdate > this.recordlist[i - 1] && e.creationdate < this.recordlist[i]) {
                                        //     temp.qty = Number(e.record)
                                        temp.record = Number(e.record)
                                        temp.added = Number(e.added)
                                        temp.seqname = e.datatype + "/(" + e.unit + ")"
                                    }
                                })
                                result.push(temp)
                            }

                        }
                        else {
                            values.seq.map(a => {
                                for (var i = 1; i <= this.recordlist.length; i++) {
                                    var temp = {};
                                    temp.name = this.recordlist[i - 1]
                                    temp.seq = a;
                                    res.data.map(e => {
                                        if (e.seq == a) {
                                            if (e.creationdate > this.recordlist[i - 1] && e.creationdate < this.recordlist[i]) {
                                                //     temp.qty = Number(e.record)
                                                temp.record = Number(e.record)
                                                temp.added = Number(e.added)
                                                temp.seqname = e.datatype + "/(" + e.unit + ")"
                                            }
                                        }
                                    })
                                    result.push(temp)
                                }
                            })
                        }
                        this.chart_data = result
                    })
            },
          
        },
    }
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
        float: right;
    }
</style>