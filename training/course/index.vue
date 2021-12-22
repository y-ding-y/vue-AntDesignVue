<template>
    <ef-main :isMainPage="true" :loading="loading" :form-items="formItems" @submit="submit">

        <a-drawer width="640" placement="right" :closable="false" :visible="visible" @close="onClose">
            <h1 v-if="actionstatus=='add'"> {{$t('add')}}</h1>
            <h1 v-else-if="actionstatus=='update'"> {{$t('update')}}</h1>
            <a-form :label-col="{ span: 5 }" :wrapper-col="{ span: 19 }">
                <a-form-item :label="$t('name')">
                    <a-input v-model="name" style="width: 100%" :placeholder="$t('name')" />
                </a-form-item>
                <a-form-item :label="$t('sort')">
                    <a-select v-model="sort">
                        <a-select-option v-for="(item,index) in sortlist" :value="item.key" :key="index"
                            :title="item.name">{{item.name}}
                        </a-select-option>
                    </a-select>
                </a-form-item>
                <a-form-item :label="$t('dept')">
                    <a-input style="width: 100%" disabled v-model="deptname" />
                </a-form-item>
                <a-form-item :label="$t('note')">
                    <a-textarea v-model="note" :rows="4" :placeholder="$t('note')" />
                </a-form-item>
                <a-form-item :label="$t('files')">
                    <a-upload :action="fileapi + '/file/upload'" @change="handleChange"
                        :lisType="actionstatus=='add'?'picture':'text'" :file-list="fileList">
                        <a-button type="primary">
                            <a-icon type="upload" /> Upload
                        </a-button>
                    </a-upload>
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

            <template slot="files" slot-scope="text,rec">
                <div style="display: flex;flex-direction: column;">
                    <div v-for="(item,index) in text.split(',')" :key="'a'+index">
                        <!-- {{ rec.files_name && (rec.files_name.split('*')[index]).split('.')}} -->
                        <transition
                            v-if="rec.file_names && ['jpg','jpeg','png','pdf'].includes((rec.file_names.split('*')[index]).split('.')[1])">
                            <img :src="icons+'/file/'+item+'/small' " :large="icons+'/file/'+item+'/large'" class="img"
                                :preview="item">
                        </transition>

                        <a v-else-if="rec.file_names" :href="icons+'/file/'+item ">{{rec.file_names.split('*')[index]}}
                        </a>
                        <span v-else>{{item}} </span>
                    </div>
                </div>
            </template>

            <template slot="actions" slot-scope="text, rec,index">
                <a-button :class="'warning'" size="small" @click="handle_update(rec,index)">
                    <a-icon type="edit" />
                </a-button>
                &nbsp;
                <a-popconfirm title="Are you sure delete ?" ok-text="Yes" cancel-text="No"
                    @confirm="handle_delete(rec)">
                    <a-button type="danger" size="small" ghost>
                        <a-icon type="delete" />
                    </a-button>
                </a-popconfirm>
            </template>
        </a-table>
        <br>
        <a-button block type="primary" @click="showDrawer" ghost>
            {{$t('add')}}
        </a-button>

    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'
    import EfMain from "@/components/page/main";
    import { merge } from '@/utils/util'
    import { mapState } from 'vuex'

    //厂别，投诉日期，投诉类别，投诉人，机种，问题描述，原因类别，上传文件
    const formItems = [
        { key: 'name', type: 'input', },
    ]
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
                formItems,
                loadcount: 0,
                sSize: 'middle',
                tableData: [],
                visible: false,
                id: "",
                name: "",
                note: "",
                deptid: "",
                files: "",
                sort: "",
                actionstatus: "",
                values: {},
                sortlist: [],
                fileList: [],
                deptname: "",
            };
        },

        computed: {
            icons() {
                return process.env.VUE_APP_API_WEB_URL;
            },
            fileapi() {
                return process.env.VUE_APP_API_BASE_URL
            },
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
                    { key: "deptname", width: 100 },
                    { key: "sortname", width: 100, },
                    { key: "coursename", width: 150, },
                    { key: "note", width: 150, },
                    { key: "files", },
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
            this.deptname = this.$props.dept;
            this.loadcount = 1;
            reqPost('PARA_SET',
                { paras: [{ "key": "para.221" }, { "key": "training.dept" }] }).then(res => {
                    this.loadcount--;
                    this.sortlist = res.data[0].data;
                    res.data[1].data.some(e => {
                        if (e.name == this.deptname) {
                            this.deptid = e.key
                            return true;
                        }
                    });

                    this.handle_get();
                })
        },
        methods: {
            handleChange(info) {
                this.fileList = [...info.fileList];
            },

            handle_get() {
                this.loadcount = 1;

                reqPost('TRAINING', { prop: "getcourse", dept: this.deptid, name: this.values.name }).then(res => {
                    this.loadcount--;
                    this.tableData = res.data;
                })
            },

            submit(values) {
                this.values = { ...values }
                this.handle_get()
            },


            showDrawer() {
                this.name = ""; 
                this.sort = "";
                this.actionstatus = "add";
                this.note = "";
                this.fileList = [];
                this.visible = true;
            },
            onClose() {
                this.visible = false;
            },

            handle_update(e) {
                this.actionstatus = "update";
                this.visible = true;
                this.name = e.dept;
                this.note = e.note
                this.sort = e.sort
                this.id = e.id;

                this.fileList = e.files.split(',').map((a, index) => {
                    return {
                        uid: index,
                        name: (e.file_names && e.file_names.split('*')[index]) || "",
                        status: 'done',
                        url: process.env.VUE_APP_API_WEB_URL + '/file/' + a,
                        id: a,
                        response: {
                            code: 200,
                            data: "1",
                            id: a,
                            message: "文件上传成功",
                            name: (e.file_names && e.file_names.split('*')[index]) || "",
                        }
                    }
                })
            },

            handle_action() {
                this.handle_add();
            },

            handle_add() {
                var fileuid = "";
                var filename = "";
                console.log(this.fileList)
                this.fileList.map((e, index) => {
                    if (index != this.fileList.length - 1) {
                        fileuid = fileuid + e.response.id + ",";
                        filename = filename + e.response.name + "*";
                    }
                    else {
                        fileuid = fileuid + e.response.id;
                        filename = filename + e.response.name;
                    }
                })

                var params = {
                    prop: this.actionstatus == 'add' ? "insertcourse" : "updatecourse",
                    name: this.name,
                    dept: this.deptid,
                    sort: this.sort,
                    files: fileuid,
                    note: this.note,
                    file_names: filename,
                    id: this.id,
                }
                console.log(params)
                if (this.name != "" && this.deptid != "" && this.sort != "") {
                    reqPost('TRAINING', params).then(res => {
                        this.loadcount--;
                        if (res.data > 0) {
                            this.$message.success("ADD Success");
                            this.handle_get();
                            this.onClose();
                        }
                        else {
                            this.$message.error("ADD Error");
                        }
                    })
                }
                else {
                    this.$message.error("課程名稱、部門、類別不能為空")
                }

            },
            handle_delete(e) {
                this.loadcount = 1;
                reqPost('TRAINING', { prop: "deletecourse", id: e.id }).then(res => {
                    this.loadcount--;
                    if (res.data > 0) {
                        this.$message.success("Delete Success");
                        this.handle_get();
                    }
                    else {
                        this.$message.error("Delete Error");
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