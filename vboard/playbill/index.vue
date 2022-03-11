<template>
    <ef-main :isMainPage="true" :loading="loading" :dataTableId="tableId" @adds="handle_add" :adds="true" :key="formi"
        :form-items="searchform" @submit="handle_search">
        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
            :scroll="scroll" :data-source="tabledata" :rowClassName="rowClassName" @change="onTableChange" rowKey="id">
            <template slot="no" slot-scope="text, rec,index">
                {{index+1}}
            </template>

            <template slot="actions" slot-scope="text, rec,index">
                <a-button :class="'warning'" size="small" @click="handle_update(rec)">
                    <a-icon type="edit" />
                </a-button>
                &nbsp;
                <a-tooltip title="編輯節目單內容">
                    <a-button :class="'success'" size="small" @click="handle_toplaylist(rec)">
                        <a-icon type="bars" />
                    </a-button>
                </a-tooltip>
                &nbsp;
                <a-popconfirm title="Are you sure delete ?" ok-text="Yes" cancel-text="No"
                    @confirm="handle_delete(rec,index)">
                    <a-button type="danger" size="small" ghost>
                        <a-icon type="delete" />
                    </a-button>
                </a-popconfirm>
            </template>
        </a-table>

        <a-drawer :width="dialogwidth" placement="right" :closable="false" :visible="visible" @close="onClose">
            <v-form :loading="loading" ref="formchild" :title="$t(actionstatus)" :form-items="formItems"
                @submit="submit" :rules="rules" :form="form">
            </v-form>
        </a-drawer>
    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'
    import { merge } from '@/utils/util'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    import VForm from '@/components/page/drawform'

    const rules = {
        name: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
        timefrom: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
        timeto: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
        channel: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
    }

    const form = {
        timefrom: ['12', '00'],
        timeto: ['12', '00'],
        name: "",
        channel: "",
    }


    const i18n = require('./i18n')
    export default {
        name: 'vboardplaybill',
        components: { EfMain, VForm },
        i18n: merge(require('@/i18n'), i18n),
        props: {
            dept: { type: String, default: "" },
        },
        data() {
            return {
                tableId: 'vboardchannel' + new Date().format('hiu'),
                loadcount: 0,
                tabledata: [],
                sSize: 'middle',
                actionstatus: "",
                id: 0,
                rules, form,
                visible: false,
                axiosprop: { "add": "insertplaybill", "update": "updateplaybill" },
                channels: [],
                formi: 0
            };
        },
        watch: {
            dept() {
                this.id++;
            },
        },
        computed: {
            searchform() {
                var arr = [{
                    key: 'channel',
                    type: 'select',
                    options: this.channels,
                }];
                return arr;
            },
            plant() {
                return this.$route.fullPath.split('/')[3]
            },
            options() {
                var hours = [];
                var mins = [];
                for (var i = 0; i < 24; i++) {
                    if (i < 10) {
                        hours.push({ value: "0" + i, label: "0" + i })
                    }
                    else {
                        hours.push({ value: i, label: i })
                    }
                }

                for (var m = 0; m < 60; m++) {
                    if (m < 10) {
                        mins.push({ value: "0" + m, label: "0" + m })
                    }
                    else {
                        mins.push({ value: m, label: m })
                    }
                }
                hours.map(e => {
                    e.children = mins
                    return e
                })
                return [hours, mins]
            },
            formItems() {
                var arr = [
                    { key: 'channel', title: this.$t('channel'), type: 'select', options: this.channels },
                    { key: 'name', title: this.$t('name'), type: 'input' },
                    { key: 'timefrom', title: this.$t('timefrom'), type: 'selecttime', options: this.options },
                    { key: 'timeto', title: this.$t('timeto'), type: 'selecttime', options: this.options },
                ]
                return arr
            },
            icons() {
                return process.env.VUE_APP_API_WEB_URL;
            },
            dialogwidth() {
                return document.documentElement.clientWidth / 2;
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { x: 1100, y: this.pageMinHeight - 200 }
            },
            loading() {
                return this.loadcount > 0;
            },
            columns() {
                return [
                    {
                        key: 'no', align: 'center', title: this.$t('no'), width: 100,
                    },
                    { key: 'channelname', align: 'left', title: this.$t('channel') },
                    { key: 'name', align: 'left', title: this.$t('name') },
                    { key: 'timefrom', align: 'left', title: this.$t('timefrom') },
                    { key: 'timeto', align: 'left', title: this.$t('timeto') },
                    { key: 'actions', align: 'left', title: this.$t('actions'), },
                    //		{ key: 'qty', align: 'right', title: this.$t("qty") },
                ].map((e) => {
                    e.dataIndex = e.key
                    //  e.sorter = ['no', 'actions'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? (function () {
                        var temp = { customRender: e.slot ?? e.key, }
                        e.key == 'no' ? (function () {
                            temp.filterDropdown = 'filterDropdown'
                            temp.filterIcon = 'filterIcon'
                        })() : (function () {

                        })()
                        return temp
                    })()
                    return e
                })
            },
        },
        created() {
            this.form.plant = this.plant
            this.loadcount = 1;
            reqPost("VBOARD", { prop: "getchannel" }).then(res => {
                this.loadcount--;
                this.channels = res.data.map(e => {
                    return {
                        key: e.id,
                        title: e.name,
                    }
                });
                this.formi++;
                this.handle_get();
            })


        },
        methods: {
            handle_toplaylist(item) {
                var temp = this.$route.fullPath.split('/')
                this.$router.push("/" + temp[1] + "/" + temp[2] + "/" + this.plant
                    + "/playcontent?playbill=" + item.id
                    + "&channel=" + item.channel
                    + "&time=" + item.timefrom + "-" + item.timeto
                    + "&name=" + item.name
                    + "&channelname=" + item.channelname)
            },
            handle_search(values) {
                this.loadcount = 1;
                reqPost("VBOARD", { prop: "getplaybill", channel: values.channel }).then(res => {
                    this.loadcount--;
                    this.tabledata = res.data;
                })
            },

            handle_get() {
                this.loadcount = 1;
                reqPost("VBOARD", { prop: "getplaybill" }).then(res => {
                    this.loadcount--;
                    this.tabledata = res.data;
                })
            },

            onClose() {
                this.visible = false;
                this.$refs.formchild.clearForm();
            },

            submit(values) {
                var fm = Number(values.timefrom[0]) * 60 + Number(values.timefrom[1])
                var to = Number(values.timeto[0]) * 60 + Number(values.timeto[1])
                if (fm == to) {
                    this.$message.error("起始時間不能相同")
                }
                else if (fm > to) {
                    this.$message.error("開始時間不能大於結束時間")
                }
                else {
                    var paras = {
                        ...values,
                        ...{ timefrom: values.timefrom[0] + ":" + values.timefrom[1] },
                        ...{ timeto: values.timeto[0] + ":" + values.timeto[1] },
                        ...{ prop: this.axiosprop[this.actionstatus] },
                        ...{ id: form.id }
                    }
                    reqPost("VBOARD", paras).then(res => {
                        if (res.data > 0) {
                            var that = this
                            this.$message.success("Success")
                            this.$refs.formchild.clearForm();
                            this.handle_get();
                            this.actionstatus == "add" ?
                                (function () {
                                    // that.$nextTick(() => {
                                    //     form.seq = that.tabledata.length + 1;
                                    // })
                                }()) :
                                (this.actionstatus == "update" ?
                                    (function () { that.visible = false; }()) :
                                    "")
                        }
                        else {
                            this.$message.error("Error")
                        }
                    })
                }
            },

            handle_delete(rec, index) {// 
                reqPost("VBOARD", { prop: "deleteplaybill", id: rec.id }).then(res => {
                    if (res.data > 0) {
                        this.$message.success("Delete Success")
                        this.tabledata.splice(index, 1);
                    }
                    else {
                        this.$message.error("Delete ERROR")
                    }
                })
            },

            handle_update(rec) {
                this.actionstatus = "update";
                this.visible = true;
                this.$nextTick(() => {
                    form.name = rec.name
                    form.id = rec.id
                    form.channel = rec.channel;
                    form.timefrom = rec.timefrom.split(":")
                    form.timeto = rec.timeto.split(":")
                })
            },

            handle_add() {
                this.actionstatus = "add";
                this.visible = true;
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

    .img {
        width: 2rem;
        height: 2rem;
        border-radius: .04rem;
        margin-left: .3rem;
        margin-top: .3rem;
        object-fit: cover;
    }
</style>