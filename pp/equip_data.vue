<template>
    <ef-main :id="imageId" :isMainPage="true" title="" :loading="loading" :openSubmit="true" :form-items="formItems"
        :dataTableId="tableId" :imageDomId="imageId" @submit="submit" @resize="resize">

        <!-- <a-table :columns="columns1" :data-source="data" :expanded-row-keys="expandedRowKeys" @expand="onExpand" /> -->

        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns"
            :pagination="{ pageSize }" :data-source="tableData" @change="onTableChange" rowKey="key" :indentSize="25"
            :scroll="scroll">


            <span v-for="item in cols" :slot="item.key+'Title'" :key="item+'Title'">
                <h4 style=" text-overflow: ellipsis;
                              overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                    {{ item.name }}
                </h4>
            </span>

            <template slot="no" slot-scope="txt, rec ,index">
                <a-tooltip>
                    <template slot="title">
                        {{ $t('name')}}： {{rec.name}}<br> {{$t('addr')}}：{{rec.addr}}<br>
                        {{ $t('ip')}}：{{rec.ip}}<br> key：{{rec.key}}
                    </template>
                    <div style="font-weight:800; text-overflow: ellipsis;
                    overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                        <a @click="onOpenList(rec)">{{ (currentPage - 1) * pageSize +index + 1 }}.&nbsp;{{rec.name}}</a>

                    </div>
                </a-tooltip>
            </template>
        </a-table>

        <v-equip v-if="visible" :key="vkey" :paras="paras" :visible="visible" :action-fix="'equip_date'">
        </v-equip>

    </ef-main>
</template>

<script>

    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    import { merge, } from '@/utils/util'
    import bus from './bus.js'
    import VEquip from './equip_dialog.vue'
    const formItems = [
        {
            key: 'datetime',
            dataIndex: 'iTime',
            type: 'datetime',
            format: 'YYYY-MM-DD HH:mm',
            allowClear: false,
            value: new Date().format('Y-m-d h:i'),
        },
        {
            key: 'type',
            type: 'select',
            paraKey: 'SfcsPp.TYPE',
            value: "101"
        },
        {
            key: 'building',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsPp.GTKBUILDING',
        },
        {
            key: 'floor',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsPp.GTKFLOOR',
        },
    ]


    export default {
        name: 'equip_date',
        i18n: merge(require('@/i18n'), require('./i18n')),
        components: { EfMain, VEquip },
        data() {
            return {
                imageId: 'fpyimage' + new Date().format('hiu'),
                tableId: 'fpytable' + new Date().format('hiu'),
                loadcount: 0,
                sSize: 'middle',
                formItems,
                vkey: 0,
                i: 0,
                i_detail: 0,
                sortOrder: '',
                footerRowKey: '',
                tableData: [],
                currentPage: 1,
                pageSize: 100,
                paras: {},
                visible: false,
                formValues: {},
                cols: [],
                cols_first: [],
                distdata: [],

                depts: [],
                equips: [],


                expandedRowKeys: [],
            }
        },
        computed: {
            scroll() {
                if (this.formValues.type == "101") {
                    return { x:this.cols.length * 150 + 100, y: this.pageMinHeight - 200 }
                }
                else {
                    return { x: 2 * 150 + 100, y: this.pageMinHeight - 200 }
                }
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),
            // title() {
            //     return this.$t('detailTitle')
            // },
            loading() {
                return this.loadcount > 0
            },

            columns() {
                var arr = [
                    {
                        key: 'no', align: 'left', width: 200, title: this.$t('no'), fixed: "left",
                        scopedSlots: { customRender: 'no' },
                    },
                    { key: 'floorname', dataIndex: "floorname", align: 'left', width: 90, title: this.$t('floorname'), fixed: "left", },
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
                        //   title: a.name.replace(a.name.split(" ")[0], ""),
                        dataIndex: a.seq,
                        align: "right",
                        title: a.datatype + "/(" + a.unit + ")",
                        slots: { title: a.seq + 'Title' },
                        scopedSlots: { customRender: a.slot ?? a.seq }
                    })
                })

                temp = ((this.formValues.type == "101") ? [...arr, ...temp] : [...arr, ...cols_warm])
                
                return temp;
            },
        },
        watch: {},
        created() {

            bus.$on("close", msg => {
                this.visible = msg._visible;
            })
        },
        mounted() { },
        methods: {

            submit(values) {
                this.tableData = [];
                this.depts = [];
                this.equips = [];
                this.formValues = { ...values } 
                if (values.type == "101") {
                    this.handle_cols();
                    this.loadcount = 3
                }
                else {
                    this.loadcount = 2
                }
                this.handle_search(values)
            },
            handle_cols() {
                reqPost("SFCS_PP", {
                    prop: "attr", order: "seq", orgId: "476",
                    equip: "9186" || "9199", type: "101",
                    select: "DISTINCT seq as SEQ,unit,dataType AS DATATYPE"
                }).
                    then(res => {
                        this.loadcount--;
                        this.cols = res.data;
                    })
            },

            handle_search(values) {
                reqPost("SFCS_PP", { ...values, prop: "equip", orgId: "476", order: "floorName" })
                    .then(res => {
                        this.loadcount--;
                        this.equips = res.data.map(e => {
                            return {
                                key: e.key,
                                name: e.name,
                                description: e.description,
                                statusname: e.statusname,
                                ip: e.ip,
                                addr: e.addr,
                                floorname: e.floorname,
                                areaname: e.areaname,
                                attrcount: e.attrcount,
                            }
                        })
                        this.handle_data(values);
                    })
            },

            // //数组去重
            // dedupe(array) {
            //     return Array.from(new Set(array));
            // },


            handle_data(values) {
                var params = {
                    prop: "data",
                    type: values.type,
                    orgId: "476",
                    //  latest: "0.04",
                    time: values.iTime,
                    select: "record as RECORD,unit AS UNIT,seq AS SEQ,equip AS EQUIP",
                };
                reqPost("SFCS_PP", params)
                    .then(res => {
                        this.loadcount--;
                        this.tableData = this.equips.map(e => {
                            res.data.map(a => {
                                if (e.key === a.equip) {
                                    e[a.seq] = Number(a.record)
                                }
                            })
                            return e;
                        })
                    })
            },

            onOpenList(record) {
                this.vkey++;
                this.visible = true;
                this.paras = { equip: record.key, type: this.formValues.type, equipname: record.name }
            },

            resize(size) {
                this.sSize = size
            },
            onTableChange(pagination, filters, sorter) {
                this.sortOrder = sorter.order
                this.currentPage = pagination.current
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