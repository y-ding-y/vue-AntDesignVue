<template>
    <ef-main :isMainPage="true" :loading="loading">

        <a-drawer width="640" placement="right" :closable="false" :visible="visible" @close="onClose">
            <h1 v-if="actionstatus=='add'"> {{$t('add')}}</h1>
            <h1 v-else-if="actionstatus=='update'"> {{$t('update')}}</h1>
            <a-form :label-col="{ span: 5 }" :wrapper-col="{ span: 19 }">
                <a-form-item :label="$t('dept')">
                    <a-input v-model="name" style="width: 100%" :placeholder="$t('dept')" />
                </a-form-item>
                <a-form-item :label="$t('msgto')">
                    <a-input v-model="msgto" :placeholder="$t('placeholdermsgto')" />
                </a-form-item>
                <a-form-item :label="$t('note')">
                    <a-textarea v-model="note" :rows="4" :placeholder="$t('note')" />
                </a-form-item>
            </a-form>
            <div style="text-align: right;">
                <a-button :class="'success'" @click="handle_action()">
                    <a-icon type="check" /> OK
                </a-button>
            </div>
        </a-drawer>

        <a-table class="basebg" :size="sSize" :bordered="true" :columns="columns" :data-source="tableData"
            :pagination="false" :indentSize="25" rowKey="id" :scroll="scroll">
            <template slot="no" slot-scope="txt, rec, i">
                {{ i + 1 }}
            </template>

            <template slot="actions" slot-scope="text, rec,index">
                <a-button :class="'warning'" size="small" @click="handle_update(rec,index)">
                    <a-icon type="edit" />
                </a-button>
                <!-- &nbsp;
                <a-popconfirm title="Are you sure delete ?" ok-text="Yes" cancel-text="No"
                    @confirm="handle_delete(rec)">
                    <a-button type="danger" size="small" ghost>
                        <a-icon type="delete" />
                    </a-button>
                </a-popconfirm> -->
            </template>
        </a-table>

    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'
    import EfMain from "@/components/page/main";
    import { merge } from '@/utils/util'
    import { mapState } from 'vuex'

    //厂别，投诉日期，投诉类别，投诉人，机种，问题描述，原因类别，上传文件

    const i18n = require('./i18n')
    export default {
        name: 'trainingdept',
        components: { EfMain },
        i18n: merge(require('@/i18n'), i18n),
        props: {
            dept: { type: String, default: "" },
        },
        data() {
            return {
                loadcount: 0,
                sSize: 'middle',
                tableData: [],
                visible: false,
                id: "",
                name: "",
                note: "",
                msgto: "",
                actionstatus: "",
                values: {},
            };
        },

        computed: {
            scroll() {
                return { x: 10, y: this.pageMinHeight - 200 }
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),

            loading() {
                return this.loadcount > 0;
            },

            columns() {
                var arr = [
                    { key: 'no', width: 90, align: "center" },
                    { key: "name", width: 250 },
                    { key: "msg_to", width: 100, },
                    { key: "note", },
                    { key: "actions", width: 180, }
                    //,sorter: (a, b) => a.complaindate > b.complaindate ? 1 : -1
                ].map((e) => {
                    e.dataIndex = e.key
                    e.title = this.$t(e.key)
                    e.slots = { title: e.key + 'Title' }
                    e.scopedSlots = { customRender: e.slot ?? e.key }
                    return e
                })
                return arr;
            },
        },
        created() {
            this.handle_get();
        },
        methods: {
            handle_get() {
                this.loadcount = 1;

                reqPost('TRAINING', { prop: "getdept", dept: this.$props.dept }).then(res => {
                    this.loadcount--;
                    this.tableData = res.data;
                })
            },


            showDrawer() {
                this.name = "";
                this.msgto = "";
                this.note = "";
                this.actionstatus = "add";
                this.visible = true;
            },
            onClose() {
                this.visible = false;
            },

            handle_update(e) {
                this.actionstatus = "update";
                this.visible = true;
                this.name = e.name;
                this.note = e.note
                this.msgto = e.msg_to;
                this.id = e.id;
            },

            handle_action() {
                if (this.actionstatus == 'add') {
                    this.handle_add();
                }
                else if (this.actionstatus == 'update') {
                    this.handle_saveupdate();
                }
            },
            handle_saveupdate() {
                var params = {
                    prop: "updatedept",
                    id: this.id,
                    name: this.name,
                    note: this.note,
                    msgto: this.msgto
                }
                reqPost('TRAINING', params).then(res => {
                    this.loadcount--;
                    if (res.data > 0) {
                        this.$message.success("Update Success");
                        this.handle_get();
                        this.onClose();
                    }
                    else {
                        this.$message.error("Update Error");
                    }
                })
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