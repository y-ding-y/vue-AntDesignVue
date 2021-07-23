<template>
    <ef-main :id="imageId" :isMainPage="true" :title="$t('snTitle')" :loading="loading" :form-items="formItems"
        :imageDomId="imageId" :i18n="i18n" @submit="submit">

        <a-tabs default-active-key="activedsn" tab-position="left" @change="handle_chosesn" :key="tabkey">
            <a-tab-pane v-for="(item,index) in snlist" :key="index" :tab="item">
                <a-spin :spinning="listLoading">
                    <a-collapse v-model="activeKey">
                        <a-collapse-panel v-for="item in collapselist" :key="item.key" :header="item.name"
                            :showArrow="false" :style="customStyle">
                            <v-form :keys="item.lists" :valuelist="valuelist" :action-fix="'sfcsSysSnView'"></v-form>
                        </a-collapse-panel>
                    </a-collapse>
                </a-spin>


                <a-card style="width:100%" :tab-list="tabList" :active-tab-key="key" :headStyle="heads"
                    @tabChange="key => onTabChange(key, 'key')">
                    <v-table :sn="tablesn" :types="tabletype" :key="tablekey" :cols="tablelist" :order="tableorder"
                        :api="tableapi" :action-fix="'sfcsSysSnView'">
                    </v-table>
                </a-card>
            </a-tab-pane>
        </a-tabs>

    </ef-main>
</template>

<script>

    import EfMain from '@/components/page/main'
    import { merge } from '@/utils/util' // saveToExcel 
    import { reqPost } from '@/services/base'
    import VForm from './form_component.vue'
    import VTable from './table_component.vue'

    const sn = ["sn", "productsn", "turninorderno", "statusname", "assysn",
        "odissw", "lancode", "repairedcounter", "mbsn", "shipno",
        "imei", "meid", "coa", "shiptime", "revisionno"];
    const wo = ["wo", "targetqty", "familyname", "hdiversion",
        "pn", "hdipn", "sysbios", "kbcbios", "pndesc"];
    const group = ["lot", "palletno"];
    const ship = ["sokey", "oempn"];
    const line = ["linename", "wiproutecodename", "sitename", "lastroutecodename", "routename"];

    const collapselist = [
        {
            key: "sn",
            name: "流程卡资讯",
            lists: sn,
        },
        {
            key: "wo",
            name: "工单资讯",
            lists: wo,
        },
        {
            key: "group",
            name: "Group Info",
            lists: group
        },
        {
            key: "ship",
            name: "销售单资讯",
            lists: ship
        },
        {
            key: "line",
            name: "产线讯息",
            lists: line
        }
    ]
    const i18n = require('./i18n')
    export default {
        i18n: merge(require('@/i18n'), i18n),
        components: { EfMain, VForm, VTable },
        name: "sfcsSysSnView",
        data() {
            return {
                loadcount: 0,
                listLoading: false,
                valuelist: {},
                searchitem: {},
                snlist: [],
                activedsn: "",
                keypartssn: "",
                temp_mb: "",
                //    tablesn: "",
                tabkey: 0,

                i18n,
                imageId: 'defectimage' + new Date().format('hiu'),

                collapselist,
                text: "loading",
                activeKey: ['sn', 'wo', 'group', 'ship', 'line'],
                customStyle: "text-align:center;font-weight:blod",

                tablekey: 0,
                tabList: [
                    { key: 'defect', tab: '机台 不良品&原因', },
                    { key: 'mb', tab: 'MB 不良品&原因', },
                    { key: 'keyparts', tab: 'Key component', },
                    { key: 'snlog', tab: '货品过站情形观察', },
                ],
                key: 'defect',
                tabletype: "defect",
                // tableapi: "",
                // tableorder: "",
                heads: { backgroundColor: "#f7f7f7", padding: "1" },
            }
        },
        computed: {
            formItems() {
                return [
                    {
                        key: 'sn',
                        type: 'input',
                        value: "",
                        style: "width:300px;max-width:600px",
                        placeholder: this.$t("txttitle"),
                    },
                ]
            },
            loading() {
                return this.loadcount > 0
            },
            tableorder() {
                var res = "";
                switch (this.key) {
                    case "defect":
                    case "mb":
                        res = "iTime";
                        break;
                    case "snlog":
                        res = "processTime";
                        break;
                    case "keyparts":
                        res = "createDate";
                        break;
                }
                return res;
            },
            tablesn() {
                var res = "";
                switch (this.key) {
                    case "defect":
                    case "snlog":
                    case "keyparts":
                        res = this.keypartssn;
                        break;
                    case "mb":
                        res = this.temp_mb;
                        break;
                }
                return res;
            },
            tableapi() {
                var res = "";
                switch (this.key) {
                    case "defect":
                    case "snlog":
                    case "keyparts":
                        res = "SFCS_RSBU";
                        break;
                    case "mb":
                        res = "SFCS_SMT";
                        break;
                }
                return res;
            },
            tablelist() {
                var arr = [];
                switch (this.key) {
                    case "defect":
                        arr = [
                            { key: 'routeCodename', },
                            { key: 'defectname', },
                            { key: 'rootname' },
                            { key: 'repairname', },
                            { key: 'dutyname', },
                            { key: 'repairername', },
                            { key: 'itime', },
                            { key: 'otime', },
                        ].map((e) => {
                            e.dataIndex = e.key.toLowerCase()
                            e.title = this.$t(e.key)
                            e.align = 'left'
                            return e
                        })
                        break;
                    case "mb":
                        arr = [
                            { key: 'routeCodename', },
                            { key: 'defectname', },
                            { key: 'reasonname' },
                            { key: 'locationcode', },
                            { key: 'repairername', },
                            { key: 'itime', },
                            { key: 'otime', },
                        ].map((e) => {
                            e.dataIndex = e.key.toLowerCase()
                            e.title = this.$t(e.key)
                            e.align = 'left'
                            return e
                        })
                        break;
                    case "snlog":
                        arr = [
                            { key: 'processtime', align: "center", },
                            { key: 'statusname', align: "left", },
                            { key: 'routecodename', align: "left", },
                            { key: 'workername', align: "left" },

                        ].map((e) => {
                            e.dataIndex = e.key.toLowerCase()
                            e.title = this.$t(e.key)
                            return e
                        })
                        break;
                    case "keyparts":
                        arr = [
                            { key: 'keypn', align: "center", },
                            { key: 'keysn', align: "left", },
                            { key: 'keydesc', align: "left", },
                            { key: 'belongto', align: "left" },
                            { key: 'sitename', align: "left" },
                            { key: 'createdate', align: "left" },
                            { key: 'workername', align: "left" },
                        ].map((e) => {
                            e.dataIndex = e.key.toLowerCase()
                            e.title = this.$t(e.key)
                            return e
                        })
                        break;
                }
                return arr;
            },
        },

        created() {
            this.tablekey++;
            if (this.$route.query.sn) {
                this.formItems[0].value = this.$route.query.sn;
                this.searchitem.sn = this.$route.query.sn;
                this.handle_snlist(this.searchitem.sn.split(","));
            //    this.snlist = this.handle_snlist(this.searchitem.sn.split(","));
                // this.activedsn = this.snlist[0]
                // this.handle_sn(this.activedsn);
            }
            else {
                this.formItems[0].value = "";
            }
        },
        mounted() { },
        methods: {
            handle_chosesn(key) {
                this.activedsn = this.snlist[key];
                this.valuelist = {};
                this.handle_sn(this.activedsn);
            },

            submit(values) {
                this.searchitem = values;
                this.handle_snlist(this.searchitem.sn.split(","));
 
            },

            resize(size) {
                this.sSize = size
            },
            onTabChange(key, type) {
                this[type] = key;
                if (key == "mb") {
                    this.tabletype = "defect";
                }
                else {
                    this.tabletype = key;
                }

                this.tablekey++;
            },

            handle_snlist(element) {
                var arr = [];
                var params_sn = {
                    "prop": "sn",
                    "select": "sn",
                    "key": element
                };

                var params_keysn = {
                    "prop": "keyparts",
                    "keySn": element,
                    "select": "snKey,sn,keySn"
                };
                reqPost("SFCS_RSBU", params_sn).then(res => {
                    console.log(res.data)
                    element.map(e => {
                        res.data.some(a => {
                            if (e === a.sn) {
                                arr.push(e)
                                return true;
                            }
                        })
                    })
                    reqPost("SFCS_RSBU", params_keysn).then(res => {
                        console.log(res.data)
                        element.map(e => {
                            res.data.some(a => {
                                if (e === a.key_sn) {
                                    arr.push(e);
                                    return true;
                                }
                            })
                        })
                        console.log(arr)

                    })
                })
                this.snlist = arr;
                console.log(this.snlist)
                setTimeout(() => {
                    this.activedsn = this.snlist[0];
                    this.tabkey++;
                    console.log(this.activedsn)
                    this.handle_sn(this.activedsn);
                }, 2000);

            },

            handle_sn(element) {
                this.tablekey++;
                var params = {
                    prop: "sn",
                    key: element
                }

                this.listLoading = true;
                params.select = {
                    "SN": "sn", "PN": "pn", "WO": "wo", "FAMILYNAME": "familyName",
                    "LINENAME": "lineName", "LASTROUTECODENAME": "lastRouteCodeName",
                    "WIPROUTECODENAME": "wipRouteCodeName", "STATUSNAME": "statusName", "TIME": "time",
                    "ROUTENAME": "routeName", "IMEI": "imei", "HDIPN": "hdiPn", "HDIVERSION": "hdiVersion",
                    "TARGETQTY": "targetQty", "SOKEY": "soKey", "OEMPN": "oemPn", "SHIPTIME": "shipTime",
                    "MEID": "meid", "COA": "coa", "ASSYSN": "assySn", "mbsn": "mbSn",
                    "REPAIREDCOUNTER": "repairedCounter", "ODISSW": "odisSw", "PRODUCTSN": "productSn",
                    "PALLETNO": "palletNo", "SITENAME": "siteName", "REVISIONNO": "revisionNo",
                    "TURNINORDERNO": "turnInOrdNo", "LANCODE": "lancode", "SHIPNO": "shipNo", "LOT": "lot"
                }

                reqPost('SFCS_RSBU', params).then((res) => {
                    this.listLoading = false;
                    if (res.data.length > 0) {
                        this.valuelist = res.data[0] || "";
                        this.temp_mb = res.data[0].mbsn || "";
                        this.keypartssn = element
                        this.handle_wo(res.data[0].wo || "");
                        this.handle_bios(res.data[0].wo || "")
                    }
                    else {
                        this.handle_keyparts(element);
                    }
                })

            },

            handle_keyparts(element) {
                var params = {
                    prop: "keyparts",
                    keySn: element
                }
                this.listLoading = true;
                params.select = {
                    "SN": "sn",
                }
                reqPost('SFCS_RSBU', params).then((res) => {
                    this.listLoading = false;
                    if (res.data.length > 0) {
                        this.keypartssn = res.data[0].sn
                        this.handle_sn(res.data[0].sn)
                    }
                    else {
                        this.$message.error("Error")
                    }
                })
            },

            handle_wo(wo) {
                var params = {
                    prop: "wo",
                    wo: wo
                }
                this.listLoading = true;
                params.select = {
                    "PNDESC": "pnDesc"
                }
                reqPost('ERP_MRP', params).then((res) => {
                    this.listLoading = false;
                    this.valuelist = { ...this.valuelist, ...res.data[0] }
                })
            },

            handle_bios(wo) {
                var params = {
                    prop: "wo",
                    wo: wo
                }
                this.listLoading = true;
                params.select = {
                    "BIOS": "bios"
                }
                reqPost('SFCS_RSBU', params).then((res) => {
                    this.listLoading = false;
                    console.log(res.data);
                    var arr = res.data[0].bios.split("-")
                    if (arr.length == 2) {
                        this.valuelist = { ...this.valuelist, "sysbios": arr[0], "kbcbios": arr[1] }
                    }
                    this.valuelist = { ...this.valuelist, ...res.data[0] }
                })
            }
        },
    }
</script>