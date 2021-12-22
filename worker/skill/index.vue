<template>
    <ef-main :isMainPage="true" :loading="loading" :key="id" >
        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
            :scroll="scroll" :data-source="tabledata"  @change="onTableChange" rowKey="id">
            <template slot="no" slot-scope="txt, rec, index">
                <span> {{index+1}}</span>
            </template>

            <template slot="name" slot-scope="txt">
                <span>{{txt}}</span>
            </template>

            <template v-for="(item,index) in stationlist" :slot="item.id" slot-scope="txt,rec,i">
                <div :key="index">
                    <a-switch @change="handle_switch(rec,item.id,i)" size="small" v-model="rec[item.id].bol" />
                </div>
            </template>

        </a-table>
    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'

    import { merge } from '@/utils/util'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
 

    const i18n = require('./i18n')
    export default {
        name: 'workerSkill',
        components: { EfMain },
        i18n: merge(require('@/i18n'), i18n),
        props: {
            dept: { type: String, default: "" },
        },
        data() {
            return {
                tableId: 'workerSkill' + new Date().format('hiu'),
                loadcount: 0, 
                tabledata: [],
                sSize: 'middle',
                isedit: false,
                deptlist: [],
                deptid: "",
                deptname: "",
                actionstatus: "",
                temp_rec: {},
                stationlist: [],
                workerlist: [],
                id:0,
            };
        },
        watch: {
            dept(){ 
               this.id++;
            },
        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { x: this.columns.length * 100, y: this.pageMinHeight - 200 }
            },
            loading() {
                return this.loadcount > 0;
            },
            columns() {
                var arr = [
                    { key: 'no', align: 'center', title: this.$t('no'), width: 100 , fixed: 'left'},
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100, fixed: 'left' },
                ];

                var temp = [];
                this.stationlist.map(e => {
                    temp.push({ key: e.id, title: e.name })
                }) 
                return [...arr, ...temp].map((e) => {
                    e.dataIndex = e.key
                //    e.sorter = ['no', 'actions'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                   // e.slots = { title: this.$t(e.key) }
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
                    station: "",
                }
            }
        },
        created() {
            this.loadcount = 1;
            reqPost('WORKER', { prop: "getdeptid", name: this.$props.dept }).then(res => {
                this.loadcount--;
                this.deptid = res.data;
                this.dept_name = this.$props.dept;
                this.handle_searchstation();
            })
            // reqPost('PARA_SET', { paras: [{ "key": "WORKER.dept" }] }).then(res => {
            //     this.loadcount--;
            //     if (res.data[0].data.length > 0) {
            //         this.deptlist = res.data;
            //         this.formItems[0].value = res.data[0].data[0].key;
            //         this.deptid = res.data[0].data[0].key;
            //         this.deptname = res.data[0].data[0].name;
            //         this.handle_searchstation();
            //     }
            //     else {
            //         this.$message.error("請先維護部門")
            //     }
            // })

        },
        methods: {
            // submit(values) {
            //     this.deptid = values.dept; 
            //     this.handle_searchstation();
            // },
            handle_searchstation() {
                this.loadcount = 1;
                var params = {
                    prop: "getstation",
                    id: this.deptid
                }
                reqPost('WORKER', params).then(res => {
                    this.loadcount--;
                    this.stationlist = res.data;
                    this.handle_searchworker();
                })
            },
            handle_searchworker() {
                this.loadcount = 1;
                var params = {
                    prop: "getworker",
                    id: this.deptid
                }
                reqPost('WORKER', params).then(res => {
                    this.loadcount--;
                    this.workerlist = res.data;
                    this.handle_search();
                })
            },
            handle_search() {
                this.loadcount = 1;
                var params = {
                    prop: "getskill",
                    id: this.deptid
                }
                reqPost('WORKER', params).then(res => {
                    this.loadcount--;
                    this.tabledata = this.workerlist.map(e => {
                        this.stationlist.map(s => {
                            res.data.some(a => {
                                if (e.id == a.worker && a.station == s.id) {
                                    e[s.id] = {};
                                    e[s.id].bol = true;
                                    e[s.id].id = a.id
                                    return true;
                                }
                                else {
                                    e[s.id] = {};
                                    e[s.id].bol = false;
                                }
                            })

                        })

                        return e;
                        // res.data.map(a => {
                        //     if (e.id == a.worker) {
                        //         //  e[a.station] = true;
                        //         e[a.station] = {};
                        //         e[a.station].bol = true;
                        //         e[a.station].id = a.id;
                        //     }
                        //     else {
                        //         e[a.station] = {};
                        //         e[a.station].bol = false;
                        //         e[a.station].id = a.id;
                        //     }
                        // })

                    });  
                })
            },

            handle_switch(e, stationid, index) { 
                var params = {
                    prop: e[stationid].bol ? "addskill" : "deleteskill",
                    worker: e.id,
                    station: stationid,
                    id: e[stationid].id
                }
                reqPost("WORKER", params).then(res => {
                    if (res.data > 0) {
                        this.$message.success("Success")
                        this.tabledata.splice(index, 1, e);
                    }
                    else {
                        this.$message.error("Error")
                    }
                })
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