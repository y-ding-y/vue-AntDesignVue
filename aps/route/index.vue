<template>
    <ef-main :isMainPage="true" :loading="loading">
        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
            :scroll="scroll" :data-source="tabledata" rowKey="id">
            <template slot="no" slot-scope="txt, rec, index">
                <span v-if="!rec.isaction"> {{index+1}}</span>
                <span v-else> {{index+1}}</span>
            </template>

            <template slot="orgname" slot-scope="txt,rec">
                <span v-if=" ! rec.isaction"> {{txt}} </span>

                <div v-if="rec.isaction">
                    <a-select v-if="rec.orgidlist" v-model="rec.org_id" style="width: 100%;">
                        <a-select-option v-for="(item) in rec.orgidlist" :value="item.id" :key="item.id">{{item.name}}
                        </a-select-option>
                    </a-select>
                    <!-- <a-input v-else v-model="rec.station" style="display: none;"></a-input> -->
                </div>
            </template>

            <template v-for="(item,index) in ['deptno','product','name','name_cn','name_en']" :slot="item"
                slot-scope="txt,rec">
                <div :key="index">
                    <span v-if="! rec.isaction">{{txt}}</span>
                    <a-input v-if=" rec.isaction" v-model="rec[item]"></a-input>
                </div>
            </template>

            <!-- <template slot="name" slot-scope="txt,rec">
                <a v-if="! rec.isaction" @click="handle_todashboard(rec)">{{txt}}</a>
                <a-input v-if=" rec.isaction" v-model="rec.name" ></a-input>

            </template> -->

            <!-- <template slot="station" slot-scope="txt,rec">
                <span v-if="! rec.isaction">{{rec.station_name}}</span>
                <div v-if="rec.isaction">
                    <a-select v-if=" actionstatus=='update' && rec.stationlist" v-model="rec.org_id"
                        style="width: 100%;" :defaultValue="rec.org_id">
                        <a-select-option v-for="(item,index) in rec.stationlist" :value="item.id" :key="index"
                            :title="item.name">{{item.name}}
                        </a-select-option>
                    </a-select>
                    
                </div> 
            </template> -->

            <template slot="dept_name" slot-scope="txt,rec">
                <span v-if="! rec.isaction">{{txt}}</span>
                <a-input v-if=" rec.isaction" v-model="rec.dept_name" disabled></a-input>
            </template>

            <template slot="annu" slot-scope="txt,rec">
                <span v-if="! rec.isaction">{{Number(txt)}}</span>
                <a-input v-if=" rec.isaction" v-model="rec.annu" disabled></a-input>
            </template>


            <template slot="actions" slot-scope="text, rec,index">
                <div v-if="!rec.isaction">
                    <a-tooltip title="編輯基本資料">
                        <a-button :class="'warning'" size="small" @click="handle_update(rec)">
                            <a-icon type="edit" />
                        </a-button>
                    </a-tooltip>

                    &nbsp;
                    <a-popconfirm title="Are you sure delete ?" ok-text="Yes" cancel-text="No"
                        @confirm="handle_delete(rec,index)">
                        <a-button type="danger" size="small" ghost>
                            <a-icon type="delete" />
                        </a-button>
                    </a-popconfirm>
                </div>

                <div v-if="rec.isaction">
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
        <a-button block type="primary" ghost @click="handle_add">Add</a-button>
    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'

    import { merge } from '@/utils/util'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'


    const i18n = require('./i18n')
    export default {
        name: 'workerInfo',
        components: { EfMain },
        i18n: merge(require('@/i18n'), i18n),
        props: {
            dept: { type: String, default: "" },
        },
        data() {
            return {
                tableId: 'oqaworkerdstation' + new Date().format('hiu'),
                loadcount: 0,
                tabledata: [],
                sSize: 'middle',
                isedit: false,
                deptlist: [],
                deptid: "",
                deptname: "",
                actionstatus: "",
                temp_rec: {},

                orgidlist: []
            };
        },
        watch: {

        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { y: this.pageMinHeight - 200 }
            },
            loading() {
                return this.loadcount > 0;
            },
            columns() {
                return [
                    { key: 'no', align: 'center', title: this.$t('no'), width: 100 },
                    { key: 'orgname', align: 'left', title: this.$t('orgname'), width: 100 },
                    { key: 'deptno', align: 'left', title: this.$t('deptno'), width: 100 },
                    { key: 'product', align: 'left', title: this.$t('product'), width: 100 },
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100 },
                    { key: 'name_cn', align: 'left', title: this.$t('name_cn'), width: 100 },
                    { key: 'name_en', align: 'left', title: this.$t('name_en'), width: 100 },
                    { key: 'actions', align: 'left', title: this.$t('actions'), width: 130 },
                    //		{ key: 'qty', align: 'right', title: this.$t("qty") },
                ].map((e) => {
                    e.dataIndex = e.key
                    //  e.sorter = ['no', 'actions'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e
                })
            },
            forms() {
                return {
                    org_id: "",
                    orgidlist: this.orgidlist,
                    deptno: "",
                    product: "",
                    name: "",
                    name_cn: "",
                    name_en: "",
                    isaction: true,
                }
            }
        },
        created() {
            this.loadcount = 1;
            // reqPost('WORKER', { prop: "getdeptid", name: this.$props.dept }).then(res => {
            //     this.loadcount--;
            //     this.deptid = res.data;
            //     this.deptname = this.$props.dept;
            //     this.handle_search();
            // }) 
            reqPost('PARA_SET', { paras: [{ "key": "PARA.201" }] }).then(res => {
                this.loadcount--; 
                if (res.data.length > 0) {
                    //  this.plantlist = res.data[0].data;
                    this.orgidlist = res.data[0].data.map(e => {
                        return {
                            id: String(e.id),
                            name: e.name
                        }
                    });
                }
                else {
                    this.$message.error("請先維護部門")
                }
            })

            this.handle_search();
        },
        methods: {

            handle_search() {
                this.loadcount = 1;
                var params = {
                    prop: "getapsroute",
                    // id: this.deptid
                }
                reqPost('APS', params).then(res => {
                    this.loadcount--;
                    this.tabledata = res.data.map(e => {
                        e.isaction = false;
                        return e;
                    });
                })
            },

            handle_delete(rec, index) {
                reqPost("APS", { prop: "deleteapsroute", id: rec.id }).then(res => {
                    if (res.data > 0) {
                        this.i++;
                        this.$message.success("Delete Success")
                        this.tabledata.splice(index, 1);
                    }
                    else {
                        this.$message.error("Delete ERROR")
                    }
                })
            },

            handle_update(rec) {
                var flag = 1;
                for (var i of this.tabledata) {
                    if (i.isaction) {
                        this.$message.warning("有編輯尚未保存")
                        flag = -1;
                    }
                }
                this.tabledata = this.tabledata.map(e => {
                    if (e.id == rec.id && flag == 1) {
                        this.temp_rec = { ...rec };
                        e.isaction = true;
                        e.orgidlist = this.orgidlist;
                        // e.org_id=e.orgidlist[0].id 
                        this.actionstatus = "update"
                    }
                    return e;
                })
            },
            handle_saveupdate(rec) {
                if (rec.station == 0) {
                    this.$message.error("Please Chose Station")
                }
                else {
                    rec.prop = "updateapsroute";
                    reqPost("APS", rec).then(res => {
                        if (res.data > 0) {
                            this.$message.success("Update Success")
                            this.handle_search();
                        }
                        else {
                            this.$message.error("Update Error")
                        }
                    })
                }
            },
            handle_add() {
                const temp = { ...this.forms }
                for (var i of this.tabledata) {
                    if (i.isaction) {
                        return this.$message.warning("请先保存当前！")
                    }
                }
                this.actionstatus = "add";
                temp.id = "id" + this.tabledata.length + 1;
                this.tabledata.push(temp)
            },
            handle_close(rec, index) {
                if (this.actionstatus == "add") {
                    this.tabledata.splice(index, 1);

                }
                else if (this.actionstatus == "update") {
                    this.tabledata.splice(index, 1, this.temp_rec);
                }
            },
            handle_saveadd(rec) {
                var item = rec;
                item.dept = this.deptid;
                item.prop = "insertapsroute";
                reqPost("APS", item).then(res => {
                    if (res.data > 0) {
                        this.$message.success("Add Success")
                        this.handle_search();
                    }
                    else {
                        this.$message.error("Add Error")
                    }
                })
            },

            clear_input(e, index) {
                var temp = { ...e }
                temp.stationlist = [];
                temp.lev_date = "";
                temp.salary_type = "";
                temp.annu_date = "";
                temp.name = "";
                this.tabledata.splice(index, 1, temp);
            },

            resize(size) {
                this.sSize = size
            },


            rowClass(record) {
                return record.key === this.footerRowKey ? 'table-footer' : null
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