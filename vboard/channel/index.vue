<template>
    <ef-main :isMainPage="true" :loading="loading" :key="id" :dataTableId="tableId" @adds="handle_add" :adds="true">
        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
            :scroll="scroll" :data-source="tabledata" :rowClassName="rowClassName" @change="onTableChange" rowKey="id">
            <template slot="no" slot-scope="text, rec,index">
                {{index+1}}
            </template>

            <template slot="plant" slot-scope="text">
                {{text.toUpperCase()}}
            </template>


            <template slot="screenpic" slot-scope="text,rec">
                <transition>
                    <img :src="icons+'/file/'+rec.screenpic+'/small' " :large="icons+'/file/'+rec.screenpic+'/large'"
                        class="img" :preview="rec.screenpic">
                </transition>
            </template>

            <template slot="actions" slot-scope="text, rec,index">
                <a-tooltip title="視頻播放">
                    <a-button :class="'success'" size="small" @click="handle_toplayer(rec)">
                        <a-icon type="video-camera" />
                    </a-button>
                </a-tooltip>

                &nbsp;
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
        screenpic: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
    }

    const form = {
        plant: "",
        name: "",
        screenpic: [],
    }


    const i18n = require('./i18n')
    export default {
        name: 'vbooardchannel',
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
                axiosprop: { "add": "insertchannel", "update": "updatechannel" },
                channels: [],
            };
        },
        watch: {
            dept() {
                this.id++;
            },
        },
        computed: {
            plant() {
                return this.$route.fullPath.split('/')[3]
            },
            formItems() {
                var arr = [
                    { key: 'plant', title: this.$t('plant'), type: 'input', disabled: true },
                    { key: 'name', title: this.$t('name'), type: 'input' },
                    {
                        key: 'screenpic', type: 'upload', title: this.$t('screenpic'),
                        picture: true, accept: ".jpg,.jpeg,.png,.gif"
                    },
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
                    { key: 'plant', align: 'left', title: this.$t('plant') },
                    { key: 'name', align: 'left', title: this.$t('name') },
                    { key: 'screenpic', align: 'left', title: this.$t('screenpic') },
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
            this.handle_get();

        },
        methods: {
            handle_get() {
                this.loadcount = 1;
                reqPost("VBOARD", { prop: "getchannel" }).then(res => {
                    this.loadcount--;
                    this.tabledata = res.data;
                })
            },

            onClose() {
                this.visible = false;
                this.$refs.formchild.clearForm();
            },

            submit(values) {
                if (values.screenpic[0].originFileObj != undefined) {
                    let formData = new FormData();
                    formData.append('file', values.screenpic[0].originFileObj);
                    this.loadcount = 1;
                    reqPost('FILE_UPLOAD', formData).then((res) => {
                        this.loadcount--;
                        if (res.code == 200) {
                            this.hanlde_action(values, res.id);
                        }
                        else {
                            this.$message.error("上傳失敗")
                        }
                    })
                }
                else {
                    this.hanlde_action(values, form.fileid);
                }
            },

            hanlde_action(values, fileid) {
                var paras = {
                    ...values,
                    ...{ fileid: fileid },
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
            },

            handle_delete(rec, index) {// 
                reqPost("VBOARD", { prop: "deletechannel", id: rec.id }).then(res => {
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
                    form.screenpic = [{
                        uid: "1",
                        name: "屏保圖片.jpg",
                        url: this.icons + '/file/' + rec.screenpic + '/large'
                    }]
                    form.name = rec.name
                    form.id = rec.id
                    form.fileid = rec.screenpic
                })
            },
            handle_toplayer(e) {
                console.log(e)
                var temp = this.$route.fullPath.split('/')
                this.$router.push("/" + temp[1] + "/" + temp[2] + "/" + this.plant
                    + "/player?screenpic=" + e.screenpic
                    + "&channel=" + e.id)
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