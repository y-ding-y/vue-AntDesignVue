 
<template>
    <ef-main :isMainPage="true" :loading="loading" :form-items="formItems" @submit="submit">
        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
            :scroll="scroll" :data-source="tabledata" rowKey="id">
            <template slot="no" slot-scope="txt, rec, index">
                <span v-if="!rec.isaction"> {{index+1}}</span>
                <span v-else> {{index+1}}</span>
            </template>

            <template slot="routename" slot-scope="txt,rec">
                <span v-if=" ! rec.isaction"> {{txt}} </span>

                <div v-if="rec.isaction">
                    <a-select v-if="rec.routelist" v-model="rec.route" style="width: 100%;">
                        <a-select-option v-for="(item) in rec.routelist" :value="item.id" :key="item.id">{{item.name}}
                        </a-select-option>
                    </a-select>
                    <!-- <a-input v-else v-model="rec.station" style="display: none;"></a-input> -->
                </div>
            </template>

            <template v-for="(item,index) in ['pn']" :slot="item" slot-scope="txt,rec">
                <div :key="index">
                    <span v-if="! rec.isaction"><b>{{txt}}</b></span>
                    <a-input v-if=" rec.isaction" v-model="rec[item]"></a-input>
                </div>
            </template>

            <template slot="description" slot-scope="text, record">
                <a-tooltip v-if="! record.isaction" :title="record.description">
                    <div style="text-overflow: ellipsis;overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                        <b>  {{ record.description }}</b>
                    </div>
                </a-tooltip>
                <a-input v-if=" record.isaction" v-model="record.description"></a-input>
            </template>

            <template slot="cn_description" slot-scope="text, record">
                <a-tooltip v-if="! record.isaction" :title="record.cn_description">
                    <div style="text-overflow: ellipsis;overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                      {{ record.cn_description }}
                    </div>
                </a-tooltip>
                <a-input v-if=" record.isaction" v-model="record.cn_description"></a-input>
            </template>


            <template slot="actions" slot-scope="text, rec,index">
                <div v-if="!rec.isaction">

                    <a-button :class="'warning'" v-if="cur_config!=''" size="small" @click="handle_update(rec)">
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
                        @click="actionstatus=='add'?handle_saveadd(rec,index):handle_saveupdate(rec)">
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
        <a-button v-if="cur_config!=''" block type="primary" ghost @click="handle_add">Add</a-button>

        <a-modal title="Add New Component" :visible="visible" :width="800" @ok="handle_saveadd" @cancel="handleCancel">
            <a-form :form="form" :label-col="{ span: 6 }" :wrapper-col="{ span: 12 }">
                <a-form-item :label="$t('pn')">
                    <a-input autocomplete="off" v-decorator="[
                    'pn',
                    { rules: [{ required: true, message: 'Please input your name' }] },
                  ]" />
                </a-form-item>
                <a-form-item :label="$t('description')">
                    <a-input autocomplete="off" v-decorator="[
                    'description',
                    { rules: [{ required: false}] },
                  ]" />
                </a-form-item>
                <a-form-item :label="$t('cn_description')">
                    <a-input autocomplete="off" v-decorator="[
                    'cn_description',
                    { rules: [{ required: false, }] },
                  ]" />
                </a-form-item>
            </a-form>
        </a-modal>

    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'

    import { merge } from '@/utils/util'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    const formItems = [
        {
            key: 'config',
            type: 'select',
            paraKey: 'APS.config',
            style: 'width:300px;mix-width:600px',
        },

    ]

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
                formItems,
                sSize: 'middle',
                isedit: false,
                deptlist: [],
                deptid: "",
                deptname: "",
                actionstatus: "",
                temp_rec: {},

                routelist: [],
                cur_config: "",
                visible: false,
                form: this.$form.createForm(this, { name: 'dynamic_rule' })
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
                    { key: 'configname', align: 'left', title: this.$t('configname'), width: 150 },
                    { key: 'pn', align: 'left', title: this.$t('pn'), width: 150 },
                    { key: 'description', align: 'left', title: this.$t('com_description'), },
                    { key: 'cn_description', align: 'left', title: this.$t('cn_description'), },
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
                    route: "",
                    routelist: this.routelist,
                    seq: 0,
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
            reqPost('APS', { prop: "getapscom" }).then(res => {
                this.loadcount--;
                if (res.data.length > 0) {
                    //  this.plantlist = res.data[0].data;
                    this.routelist = res.data.map(e => {
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

            // this.handle_search();
        },
        methods: {
            submit(values) {
                this.loadcount = 1;
                var params = {
                    prop: "getapscom",
                    config: values.config
                    // id: this.deptid
                }
                reqPost('APS', params).then(res => {
                    this.loadcount--;
                    this.cur_config = values.config ?? ""
                    this.tabledata = res.data.map(e => {
                        e.isaction = false;
                        return e;
                    });
                })
            },
            handle_search() {
                this.loadcount = 1;
                var params = {
                    prop: "getapscom",
                    config: this.cur_config
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
                reqPost("APS", { prop: "deleteapscom", id: rec.id }).then(res => {
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
                        e.routelist = this.routelist;
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
                    rec.prop = "updateapscom";
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
                this.visible = true;
                // const temp = { ...this.forms }
                // for (var i of this.tabledata) {
                //     if (i.isaction) {
                //         return this.$message.warning("请先保存当前！")
                //     }
                // }
                // this.actionstatus = "add";
                // var len = this.tabledata.length;

                // temp.seq = len > 0 ? (Number(this.tabledata[len - 1].seq) + 1) : 1
                // temp.id = "id" + this.tabledata.length + 1;
                // this.tabledata.push(temp)
            },
            handle_close(rec, index) {
                if (this.actionstatus == "add") {
                    this.tabledata.splice(index, 1);

                }
                else if (this.actionstatus == "update") {
                    this.tabledata.splice(index, 1, this.temp_rec);
                }
            },
            handle_saveadd() {
                var params = { config: this.cur_config, prop: "insertapscom" }
                this.form.validateFields((err, fieldsValue) => {
                    if (err) {
                        return err
                    }
                    reqPost("APS", { ...fieldsValue, ...params }).then(res => {
                        if (res.data > 0) {
                            this.$message.success("Add Success")
                            //this.tabledata.push(rec) 
                            this.visible = false;
                            this.form.resetFields()
                            this.handle_search();
                        }
                        else {
                            this.$message.error("Add Error")
                        }
                    })
                })

            },

            getroutelistid(element) {
                var res = "";
                this.routelist.some(e => {
                    if (e.id == element) {
                        res = e.name;
                        return true;
                    }
                })
                return res;
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

          
            handleCancel() {
                this.visible = false;
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