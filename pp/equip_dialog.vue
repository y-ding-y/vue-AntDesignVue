<template>
    <div>
        <a-modal :visible="visible" :title="$t('Detaildialog')" :width="listWidth" :footer="null"
            @cancel="handleCancel">
            <ef-main :id="imageId" :isMainPage="true" :title="title" :loading="loading" :openSubmit="true"
                @change="handle_change" :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId"
                @submit="submit" @resize="resize">
                <!-- <a-table :columns="columns1" :data-source="data" :expanded-row-keys="expandedRowKeys" @expand="onExpand" /> -->

                <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns"
                    :pagination="{ pageSize }" :data-source="tableData" @change="onTableChange" rowKey="name"
                    :scroll="scroll">

                    <span v-for="item in cols" :slot="item.key+'Title'" :key="item+'Title'">
                        <h4 style=" text-overflow: ellipsis;
                                  overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                            {{ item.name }}
                        </h4>
                    </span>

                    <template slot="no" slot-scope="txt, rec ,index">
                        {{ (currentPage - 1) * pageSize +index + 1 }}
                    </template>
                    <template slot="name" slot-scope="txt, rec ">
                        <b> {{rec.name}}</b>
                    </template>
                </a-table>
            </ef-main>
        </a-modal>
    </div>
</template>

<script>
    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import { merge, } from '@/utils/util'
    import { mapState } from 'vuex'
    import bus from './bus.js'
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
        value: ""
    },
    {
        key: 'equip',
        type: 'select',
        value: "",
        allowClear: false,
        options: [],
    },]
    export default {
        name: 'Dialog_list',
        props: {
            actionFix: String,
            visible: { type: Boolean, default: false },
            paras: { type: Object, default: () => ({}) }
        },
        components: { EfMain, },
        i18n: merge(require('@/i18n'), require('./i18n')),
        data() {
            return {
                imageId: 'fpyimage' + new Date().format('hiu'),
                tableId: 'fpytable' + new Date().format('hiu'),
                sSize: 'middle',
                loadcount: 0,
                listTableWidth: 2800,
                listWidth:
                    document.documentElement.clientWidth * 0.8 > 2000
                        ? 2000
                        : document.documentElement.clientWidth * 0.8,
                tableData: [],
                forms: {},
                equips: [],
                cols: [],
                currentPage: 1,
                pageSize: 100,
                title: "",
                options: [],
                formItems,
            }
        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { x: this.cols.length * 150 + 100, y: this.pageMinHeight - 200 }
            },
            loading() {
                return this.loadcount > 0
            },
            recordlist() {
                var temp_hour = []
                var res = []
                var days = this.GetNumberOfDays(this.forms.iTime[0], this.forms.iTime[1])
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
            columns() {
                var arr = [
                    {
                        key: 'no', align: 'left', width: 90, title: this.$t('no'), fixed: "left",
                        scopedSlots: { customRender: 'no' },
                    },
                    {
                        key: 'name', dataIndex: "name", align: 'left', width: 200, title: this.$t('name'),
                        fixed: "left",
                        scopedSlots: { customRender: "name" }
                    },
                ]
                var temp = [];
                var cols_warm = [
                    { seq: "1", unit: "°C", datatype: "溫度" },
                    { seq: "2", unit: "RH", datatype: "濕度" }
                ].map(a => {
                    return {
                        key: a.seq,
                        dataIndex: a.seq,
                        align: "right",
                        title: a.datatype + "/(" + a.unit + ")",
                        slots: { title: a.seq + 'Title' },
                        scopedSlots: { customRender: a.slot ?? a.seq }
                    }
                });
                temp = this.cols.map(a => {
                    return ({
                        key: a.seq,
                        dataIndex: a.seq,
                        align: "right",
                        title: a.datatype + "/(" + a.unit + ")",
                        slots: { title: a.seq + 'Title' },
                        scopedSlots: { customRender: a.slot ?? a.seq }
                    })
                })

                temp = ((this.forms.type == "101") ? [...arr, ...temp] : [...arr, ...cols_warm])
                return temp;
            },

        },
        created() {
            reqPost("SFCS_PP", { prop: "consts", key: "GTKEQUIP(" + this.paras.type + ")", })
                .then(res => {
                    this.options = res.data.map(e => {
                        return {
                            key: e.key,
                            title: e.name
                        }
                    });
                    this.formItems[2].options = this.options;
                })
            this.formItems[1].value = this.paras.type;
            this.title = this.paras.equipname;
            this.formItems[2].value = this.paras.equip;

        },
        watch: {
            forms(newval, oldval) {
                if (oldval.type && oldval.type != newval.type) {
                    this.handle_equip();
                }
                this.formItems[2].options.some(e => {
                    if (e.key == newval.equip) {
                        this.title = e.title;
                        return true
                    }
                });
            },
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
                this.tableData = [];
                this.loadcount = 2;
                this.forms = { ...values }
                this.handle_cols(this.forms)
            },
            handle_change(values) { 
                this.forms = { ...values };
                this.tableData = [];
            },
            handle_equip() {
                reqPost("SFCS_PP", { prop: "consts", key: "GTKEQUIP(" + this.forms.type + ")" })
                    .then(res => {
                        this.equips = res.data.map(e => {
                            return {
                                key: e.key,
                                name: e.name,
                                title: e.name,
                                description: e.description,
                                statusname: e.statusname,
                                ip: e.ip,
                                addr: e.addr,
                                floorname: e.floorname,
                                areaname: e.areaname,
                                attrcount: e.attrcount,
                            }
                        })
                        this.formItems[2].options = this.equips;
                        this.formItems[2].value = this.equips[0].key || "";
                        this.title = this.equips[0].name || ""
                    })
            },
            handle_cols(values) {
                this.cols = [];
                reqPost("SFCS_PP", {
                    prop: "attr", order: "seq", orgId: "476",
                    equip: values.equip, type: values.type,
                    select: "DISTINCT seq as SEQ,unit,dataType AS DATATYPE"
                }).
                    then(res => {
                        this.loadcount--;
                        this.cols = res.data;
                        this.handle_data(values);
                    })
            },

            handle_data(values) {
                var params = {
                    prop: "data",
                    type: values.type,
                    orgId: "476",
                    date: values.iTime,
                    equip: values.equip,
                    select: "record as RECORD,unit AS UNIT,seq AS SEQ,equip AS EQUIP,creationDate as CREATIONDATE",
                };
                reqPost("SFCS_PP", params)
                    .then(res => {
                        this.loadcount--;
                        var result = [];
                        for (var i = 1; i <= this.recordlist.length; i++) {
                            var temp = {};
                            temp.name = this.recordlist[i - 1]
                            res.data.map(e => {
                                if (e.creationdate > this.recordlist[i - 1] && e.creationdate < this.recordlist[i]) {
                                    temp[e.seq] = e.record
                                }
                            })
                            result.push(temp)
                        }

                        this.tableData = result;
                    })
            },
            resize(size) {
                this.sSize = size
            },
            onTableChange(pagination, filters, sorter) {
                this.sortOrder = sorter.order
                this.currentPage = pagination.current
            },
            handleCancel() {
                bus.$emit("close", {
                    _visible: false,
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
</style>