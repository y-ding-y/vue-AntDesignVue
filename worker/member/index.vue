<template>
    <ef-main :isMainPage="true" :loading="loading" :form-items="formItems" @submit="submit">
        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
            :scroll="scroll" :data-source="tabledata" :rowClassName="rowClassName" @change="onTableChange" rowKey="id">
            <template slot="no" slot-scope="txt, rec, index">
                <span v-if="!rec.isaction"> {{index+1}}</span>
                <span v-else> {{index+1}}</span>
            </template>

            <template slot="empno" slot-scope="txt,rec,index">
                <span v-if="! rec.isaction">{{txt}}</span>
                <a-input v-if="rec.isaction" v-model="rec.empno" @change="clear_input(rec,index)"
                    @keyup.enter="handle_searchempno(rec,index)"></a-input>
            </template>

            <template v-for="(item,index) in ['salary_type','annu_date','lev_date','name']" :slot="item"
                slot-scope="txt,rec">
                <div :key="index">
                    <span v-if="! rec.isaction">{{txt}}</span>
                    <a-input v-if=" rec.isaction" v-model="rec[item]" disabled></a-input>
                </div>
            </template>

            <template slot="station" slot-scope="txt,rec">
                <span v-if="! rec.isaction">{{rec.station_name}}</span>
                <a-select v-if=" rec.isaction" v-model="rec.station" style="width: 100%;">
                    <a-option v-for="(item,index) in rec.stationlist" :value="item.id" :key="index">{{item.name}}
                    </a-option>
                </a-select>
            </template>

            <template slot="dept_name" slot-scope="txt,rec">
                <span v-if="! rec.isaction">{{txt}}</span>
                <a-input v-if=" rec.isaction" v-model="rec.dept_name" disabled></a-input>
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

    const formItems = [
        { key: 'dept', type: 'select', paraKey: "WORKER.dept", allowClear: false, },
    ]

    const i18n = require('./i18n')
    export default {
        name: 'oqaworkerdstation',
        components: { EfMain },
        i18n: merge(require('./i18n'), i18n),
        data() {
            return {
                tableId: 'oqaworkerdstation' + new Date().format('hiu'),
                loadcount: 0,
                formItems,
                tabledata: [],
                sSize: 'middle',
                isedit: false,
                deptlist: [],
                deptid: "",
                deptname: "",
                actionstatus: "",
                temp_rec: {},
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
                    { key: 'empno', align: 'left', title: this.$t('empno'), width: 100 },
                    { key: 'dept_name', align: 'left', title: this.$t('dept'), width: 100 },
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100 },
                    { key: 'station', align: 'left', title: this.$t('station'), width: 100 },
                    { key: 'salary_type', align: 'left', title: this.$t('salary_type'), width: 100 },
                    { key: 'annu_date', align: 'left', title: this.$t('annu_date'), width: 100 },
                    { key: 'lev_date', align: 'left', title: this.$t('lev_date'), width: 100 },
                    { key: 'actions', align: 'left', title: this.$t('actions'), width: 100 },
                    //		{ key: 'qty', align: 'right', title: this.$t("qty") },
                ].map((e) => {
                    e.dataIndex = e.key
                    e.sorter = ['no', 'actions'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e
                })
            },
            forms() {
                return { 
                    name: "",
                    dept_name: this.deptname,
                    dept: "",
                    salary_type: "",
                    annu_date: "",
                    lev_date: "", 
                    station:"",
                    isaction: true,
                }
            }
        },
        created() {
            this.loadcount = 1;
            reqPost('PARA_SET', { paras: [{ "key": "WORKER.dept" }] }).then(res => {
                this.loadcount--;
                if (res.data[0].data.length > 0) {
                    this.deptlist = res.data;
                    this.formItems[0].value = res.data[0].data[0].key;
                    this.deptid = res.data[0].data[0].key;
                    this.deptname = res.data[0].data[0].name;
                    this.handle_search();
                }
                else {
                    this.$message.error("請先維護部門")
                }
            })

        },
        methods: {
            submit(values) {
                this.deptid = values.dept;
                this.handle_search();
            },

            handle_search() {
                this.loadcount = 1;
                var params = {
                    prop: "getworker",
                    id: this.deptid
                }
                reqPost('WORKER_OQA', params).then(res => {
                    this.loadcount--;
                    this.tabledata = res.data.map(e => {
                        e.isaction = false;
                        return e;
                    });
                })
            },

            handle_delete(rec, index) {
                reqPost("WORKER_OQA", { prop: "deletemember", id: rec.id }).then(res => {
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
                this.temp_rec = { ...rec };
                var flag = 1;
                for (var i of this.tabledata) {
                    if (i.isaction) {
                        this.$message.warning("有編輯尚未保存")
                        flag = -1;
                    }
                }
                this.tabledata = this.tabledata.map(e => {
                    if (e.id == rec.id && flag == 1) {
                        e.isaction = true;
                        this.actionstatus = "update"
                    }
                    return e;
                })
            },
            handle_saveupdate(rec) {
                if (rec.name == "") {
                    this.$message.error("Please input Name")
                }
                else {
                    rec.prop = "updatestation";
                    reqPost("WORKER_OQA", rec).then(res => {
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
                item.prop = "insertwowrker";
                console.log(item)
                reqPost("WORKER_OQA", item).then(res => {
                    if (res.data > 0) {
                        this.$message.success("Add Success")
                        this.handle_search();
                    }
                    else {
                        this.$message.error("Add Error")
                    }
                })
            },

            handle_searchempno(e, index) {
                console.log(e)
                var temp = { ...e }
                this.loadcount = 1;
                reqPost("WORKER_OQA", { prop: "geterpempno", empno: e.empno }).then(res => {
                    this.loadcount--;
                    console.log(res.data)
                   // temp.stationlist = this.handle_skillstation(e);
                    temp.stationlist=[];
                    temp.lev_date = res.data[0].lev_date;
                    temp.salary_type = res.data[0].salary_type;
                    temp.annu_date = res.data[0].annu_date;
                    temp.name = res.data[0].name;
                    this.tabledata.splice(index, 1, temp);
                })
            },

            handle_skillstation(e) {
                var result = [];
                this.loadcount = 1;
                reqPost("WORKER_OQA", { prop: "skillstation", id:e.id }).then(res => {
                    this.loadcount--;
                    result = res.data;
                })
                return result
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
            onTableChange(pagination, filters, sorter) {
                this.sortOrder = sorter.order
            },

            rowClass(record) {
                return record.key === this.footerRowKey ? 'table-footer' : null
            },

            sorter(a, b, rcs) {
                const order = this.sortOrder
                const fKey = this.footerRowKey

                if (a.key === fKey) {
                    return order === 'ascend' ? 1 : -1
                } else if (b.key === fKey) {
                    return order !== 'ascend' ? 1 : -1
                } else if (rcs === 'name') {
                    const aN = a[rcs]
                    const bN = b[rcs]
                    return aN === bN ? 0 : aN > bN ? 1 : -1
                } else {
                    const aI = a[rcs] ?? 1
                    const bI = b[rcs] ?? 1
                    return Math.sign(aI - bI)
                }
            },


            rowClassName(record) {
                return record.key === this.footerRowKey
                    ? 'ef-sfcs-stat-table-footer'
                    : null
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