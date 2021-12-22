<template>
    <ef-main :isMainPage="true" :loading="loading" :form-items="formItems" @submit="submit">

        <a-drawer width="640" placement="right" :closable="false" :visible="visible" @close="onClose">
            <h1 v-if="actionstatus=='add'"> {{$t('upload')}}</h1>
            <h1 v-else-if="actionstatus=='update'"> {{$t('update')}}</h1>
            <a-form :label-col="{ span: 5 }" :wrapper-col="{ span: 19 }">
                <a-form-item :label="$t('name')">
                    <a-input v-model="file_name" style="width: 100%" :placeholder="$t('file_names')" />
                </a-form-item>
                <a-form-item :label="$t('note')">
                    <a-textarea v-model="file_desc" :rows="4" :placeholder="$t('file_desc')" />
                </a-form-item>
                <a-form-item v-if="actionstatus=='add'" :label="$t('files')">
                    <a-upload :action="fileapi + '/file/upload?prop=report&useid='+folderid+'&empno='+empno"
                        @change="handleChange" list-type="picture" :file-list="fileList" :remove="handleRemove">
                        <a-button type="primary" ghost>
                            <a-icon type="upload" /> Upload
                        </a-button>
                    </a-upload>
                </a-form-item>
            </a-form>
            <div style="text-align: right;">
                <a-button :class="'success'" @click="handle_saveupdate()" :disabled="btnstatus">
                    <a-icon type="check" /> OK
                </a-button>
            </div>
        </a-drawer>

        <a-table class="basebg" :size="sSize" :bordered="true" :columns="columns" :data-source="tableData"
            :pagination="false" :indentSize="25" rowKey="id" :scroll="scroll">
            <template slot="no" slot-scope="txt, rec, i">
                {{ i + 1 }}
            </template>

            <template slot="file_name" slot-scope="text,rec">
                <a v-if="rec.file_path.split('/')[0]=='UserFiles'" :href="'http://10.87.14.166:9080/'+rec.file_path "
                    :download="text">
                    {{text.split('.')[0]}}
                </a>

                <transition v-else-if="text && ['jpg','jpeg','png','pdf'].includes(rec.file_type)">
                    <img :src="icons+'/file/'+rec.id+'/small' " :large="icons+'/file/'+rec.id+'/large'" class="img"
                        :preview="rec.id">
                </transition>
                <a v-else :href="icons+'/file/'+rec.id "> {{text.split('.')[0]}}</a>
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
            {{$t('upload')}}
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
        { key: 'keys', type: 'input', },
    ]
    const i18n = require('./i18n')
    export default {
        name: 'oqaupload',
        components: { EfMain },
        i18n: merge(require('@/i18n'), i18n),
        props: {
            folderid: { type: String, default: "1726" },
        },
        data() {
            return {
                formItems,
                loadcount: 0,
                ifrefresh: true,
                sSize: 'middle',
                i: 0,
                useridliset: [],
                tableData: [],
                visible: false,
                unit: ['Bytes', 'KB', 'MB', 'GB', 'TB', 'PB'],
                fileList: [],
                file_name: "",
                file_desc: "",
                id: "",
                fileindex: "",
                actionstatus: "",
                values: {},
            };
        },
        watch: {
            visible() {
                if (this.actionstatus == "add") {
                    this.file_name = "";
                    this.file_desc = "";
                    this.fileList = [];
                }

            },
        },
        computed: {
            btnstatus() {
                if (this.actionstatus == 'add') {
                    if (this.fileList.length > 0) {
                        return false;
                    }
                    else {
                        return true
                    }
                }
                else if (this.actionstatus == 'update') {
                    return false;
                }
                else {
                    return true
                }
            },
            empno() {
                return JSON.parse(localStorage.getItem(process.env.VUE_APP_USER_KEY)).empno
            },
            scroll() {
                return { x: 10, y: this.pageMinHeight - 200 }
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),
            fileapi() {
                return process.env.VUE_APP_API_BASE_URL
            },
            icons() {
                return process.env.VUE_APP_API_WEB_URL;
            },
            loading() {
                return this.loadcount > 0;
            },

            columns() {
                var arr = [
                    { key: 'no', width: 90, align: "center" },
                    { key: "file_name", width: 250 },
                    { key: "note", width: 200 },
                    { key: "file_size", width: 100, },
                    { key: "employee", width: 100, },
                    { key: "create_date", width: 180, },
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
                this.loadcount = 2;
                var temp_table = []
                reqPost('EF_FILE', { prop: "getfiles", folderid: this.$props.folderid, keys: this.values.keys }).then(res => {
                    this.loadcount--;
                    temp_table = res.data.map(e => {
                        this.useridliset.push(e.create_user);
                        var i = Math.floor(Math.log(e.file_size) / Math.log(1024));
                        var n = e.file_size / Math.pow(1024, i);
                        e.file_size = n.toFixed(2) + this.unit[i]
                        return e;
                    });
                    reqPost('ERP_HR_EMPLOYEE', {
                        empno: this.arr_distinct(this.useridliset),
                        select: "DISTINCT  name,empno"
                    }).then(res => {
                        this.loadcount--;
                        var model = res.data;
                        this.tableData = temp_table.map(e => {
                            model.some(a => {
                                if (a.empno == e.create_user) {
                                    e.employee = a.name;
                                    return true;
                                }
                            })
                            return e;
                        })
                    })
                })
            },

            submit(values) {
                this.values = { ...values }
                this.handle_get()
            },

            handleChange(info) {
                info.fileList = info.fileList.slice(-1);
                info.fileList.map(file => {
                    if (file.response) {
                        this.$message.success("Upload Success");
                        this.file_name = file.response.name.split('.')[0];
                        this.file_desc = "";
                        this.id = file.response.id
                    }
                });
                this.fileList = info.fileList;
            },
            handleRemove(info) {
                this.file_name = "";
                this.file_desc = "";
                this.handle_delete({ id: info.response.id })

            },

            showDrawer() {
                this.actionstatus = "add";
                this.visible = true;
            },
            onClose() {
                this.visible = false;

            },

            arr_distinct(element) {
                var resultArr = element.filter(function (item, index, self) {
                    return self.indexOf(item) == index;
                });
                return resultArr;
            },

            handle_update(e, index) {
                this.actionstatus = "update";
                this.visible = true;
                this.file_name = e.file_name.split('.')[0];
                this.file_desc = e.note
                this.id = e.id;
                this.fileindex = index;

            },

            handle_saveupdate() {
                var params = {
                    prop: "updateeffile",
                    id: this.id,
                    file_name: this.file_name,
                    description: this.file_desc,
                    empno: this.empno,
                }
                reqPost('EF_FILE', params).then(res => {
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

            handle_delete(e) {
                this.loadcount = 1;
                reqPost('EF_FILE', { prop: "deleteeffile", id: e.id }).then(res => {
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