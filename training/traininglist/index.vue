<template>
    <ef-main :isMainPage="true" :loading="loading" :form-items="formItems" @submit="submit">
        <div style="position: absolute;top:0px;left: 150px;">
            <a-radio-group v-model="viewstatus" button-style="solid">
                <a-radio-button value="table">
                    表格
                </a-radio-button>
                <a-radio-button value="calendar">
                    日曆
                </a-radio-button>
            </a-radio-group>
        </div>


        <a-drawer width="640" placement="right" :closable="false" :visible="visible" @close="onClose">
            <h1 v-if="actionstatus=='upload'"> {{$t('upload')}}</h1>
            <h1 v-else-if="actionstatus=='look'"> {{$t('look')}}</h1>
            <h1 v-else-if="actionstatus=='add'"> {{$t('add')}}</h1>
            <h1 v-else-if="actionstatus=='updateclass'"> {{$t('update')}}</h1>
            <div v-if="actionstatus=='add' || actionstatus=='updateclass'">
                <add-class :key="addi" :dept="dept" :params="params" :actionstatus="actionstatus"></add-class>
            </div>
            <div v-else>
                <a-form :label-col="{ span: 5 }" :wrapper-col="{ span: 19 }">
                    <a-form-item :label="$t('name')">
                        <a-input v-model="coursename" style="width: 100%" disabled />
                    </a-form-item>
                    <a-form-item v-if="actionstatus=='upload'" :label="$t('files')">
                        <a-upload :action="fileapi + '/file/upload'" :before-upload="beforeUpload" :file-list="fileList"
                            @change="handleChange">
                            <a-button type="primary">
                                <a-icon type="upload" /> Select File
                            </a-button>
                        </a-upload>
                    </a-form-item>
                </a-form>
                <div v-if="actionstatus=='upload'" style="text-align: right;">
                    <a-button :class="'success'" @click="handle_action()">
                        <a-icon type="check" /> OK
                    </a-button>
                </div>
                <br>
                <a-table :loading="scoreloading" class="basebg" :scroll="{  y: tableheight }" :bordered="true"
                    :columns="scorecol" :pagination="false" :data-source="tabledata" rowKey="id" size="small">
                    <template slot="no" slot-scope="txt, rec,i">
                        {{i+1}}
                    </template>
                    <template v-for="(item,index) in ['score']" :slot="item" slot-scope="txt,rec">
                        <div :key="index">
                            <span v-if="! rec.isaction">{{txt}}</span>
                            <a-input v-if=" rec.isaction" v-model="rec[item]"></a-input>
                        </div>
                    </template>


                    <template slot="pass" slot-scope="txt,rec">
                        <span v-if="! rec.isaction">
                            <a-tag v-if="txt=='Y'" color="green"> {{txt}}</a-tag>
                            <a-tag v-else-if="txt=='N'" color="red"> {{txt}}</a-tag>
                        </span>
                        <a-radio-group v-if=" rec.isaction" v-model="rec.pass" button-style="solid">
                            <a-radio-button value="Y">
                                Y
                            </a-radio-button>
                            <a-radio-button value="N">
                                N
                            </a-radio-button>
                        </a-radio-group>
                    </template>

                    <template slot="actions" slot-scope="text, rec,index">
                        <div v-if="!rec.isaction">
                            <a-button :class="'warning'" size="small" @click="handle_updatescore(rec)">
                                <a-icon type="edit" />
                            </a-button>
                            &nbsp;
                            <a-popconfirm title="Are you sure delete ?" ok-text="Yes" cancel-text="No"
                                @confirm="handle_deletescore(rec)">
                                <a-button type="danger" size="small" ghost>
                                    <a-icon type="delete" />
                                </a-button>
                            </a-popconfirm>
                        </div>
                        <div v-if="rec.isaction">
                            <a-button :class="'success'" size="small" @click="handle_saveupdatescore(rec)">
                                <a-icon type="save" />
                            </a-button>
                            &nbsp;
                            <a-button size="small" type="danger" ghost @click="handle_close(rec,index)">
                                <a-icon type="close" />
                            </a-button>
                        </div>

                    </template>

                </a-table>
            </div>
        </a-drawer>
        <div v-if="viewstatus=='table'">
            <a-table class="basebg" :size="sSize" :bordered="true" :columns="columns" :data-source="tableData"
                :pagination="false" :indentSize="25" rowKey="id" :scroll="scroll">
                <template slot="no" slot-scope="txt, rec, i">
                    {{ i + 1 }}
                </template>
                <template slot="coursename" slot-scope="txt">
                    <b>{{txt}}</b>
                </template>
                <template slot="files" slot-scope="text,rec">
                    <div style="display: flex;flex-direction: column;">
                        <div v-for="(item,index) in text.split(',')" :key="'a'+index">
                            <!-- {{ rec.files_name && (rec.files_name.split('*')[index]).split('.')}} -->
                            <transition
                                v-if="rec.file_names && ['jpg','jpeg','png','pdf'].includes((rec.file_names.split('*')[index]).split('.')[1])">
                                <img :src="icons+'/file/'+item+'/small' " :large="icons+'/file/'+item+'/large'"
                                    class="img" :preview="item">
                            </transition>
                            <a v-else-if="rec.file_names"
                                :href="icons+'/file/'+item ">{{rec.file_names.split('*')[index]}}
                            </a>
                            <span v-else>{{item}} </span>
                        </div>
                    </div>
                </template>

                <template slot="actions" slot-scope="text, rec">

                    <a-tooltip title="查看成績單" v-if="rec.record">
                        <a-button :class="'warning'" size="small" @click="handle_look(rec)">
                            <a-icon type="search" />
                        </a-button>
                    </a-tooltip>
                    <a-tooltip title="上傳成績單" v-else>
                        <a-button :class="'success'" size="small" @click="handle_upload(rec)">
                            <a-icon type="cloud-upload" />
                        </a-button>
                    </a-tooltip>
                    &nbsp;
                    <a-popconfirm title="Are you sure delete ?" ok-text="Yes" cancel-text="No"
                        @confirm="handle_delete(rec)">
                        <a-button type="danger" size="small" ghost>
                            <a-icon type="delete" />
                        </a-button>
                    </a-popconfirm>
                    &nbsp;
                    <a-tooltip title="下載成績單" v-if="rec.record">
                        <a :href="icons+'/file/'+rec.record ">
                            <a-button :class="'success'" size="small">
                                <a-icon type="vertical-align-bottom" />
                            </a-button>
                        </a>
                    </a-tooltip>

                    <a-tooltip title="修改排課" v-else-if="rec.date_to>today">
                        <a-button :class="'warning'" size="small" @click="handle_update(rec)">
                            <a-icon type="edit" />
                        </a-button>
                    </a-tooltip>

                </template>
            </a-table> <br>
            <a-button block type="primary" @click="handle_add" ghost>
                {{$t('add')}}
            </a-button>
        </div>
        <calendar v-if="viewstatus=='calendar'" :tabledate="tableData"></calendar>



    </ef-main>

</template>
<script>
    import { reqPost } from '@/services/base'
    import EfMain from "@/components/page/main";
    import AddClass from "./../trainingclass/index";
    import Calendar from "./../calendar/index";
    import { merge, } from '@/utils/util'
    import { mapState } from 'vuex'
    import { readExcel, } from './upload.js'
    import bus from './../bus.js'

    const formItems = [
        { key: 'name', type: 'input', },
    ]
    const i18n = require('./i18n')
    export default {
        name: 'trainingdept',
        components: { EfMain, AddClass, Calendar },
        i18n: merge(require('@/i18n'), i18n),
        props: {
            dept: { type: String, default: "" },
        },
        data() {
            return {
                viewstatus: "table",
                formItems,
                loadcount: 0,
                scoreloadcount: 0,
                sSize: 'middle',
                tabledata: [],
                tableData: [],
                visible: false,
                id: "",
                coursename: "",
                note: "",
                deptid: "",
                files: "",
                sort: "",
                actionstatus: "",
                values: {},
                deptlist: [],
                sortlist: [],
                fileList: [],
                course: "",
                temp_rec: {},
                score: "",
                pass: "",
                fileid: "",
                addi: 0,
                params: {},
            };
        },

        computed: {
            today() {
                return new Date().format('Y-m-d HH:mm:ss')
            },
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

            scoreloading() {
                return this.scoreloadcount > 0;
            },
            loading() {
                return this.loadcount > 0;
            },
            tableheight() {
                return this.pageMinHeight - 330;
            },
            scorecol() {
                var temp = [
                    { key: "no", width: 80 },
                    { key: "empno", width: 150 },
                    { key: "score", },
                    { key: "pass", },
                ];
                if (this.actionstatus != "upload") {
                    temp.push({ key: "actions", },)
                }
                return temp.map(e => {
                    e.dataIndex = e.key
                    e.title = this.$t(e.key)
                    e.slots = { customRender: e.key }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e;
                })
            },
            columns() {
                var arr = [
                    { key: 'no', width: 50, align: "center", fixed: "left" },
                    { key: "coursename", width: 150, fixed: "left" },
                    { key: "sortname", width: 150, },
                    { key: "place", width: 150, },
                    { key: "date_from", width: 150, },
                    { key: "date_to", width: 150, },
                    { key: "teacher", width: 100 },

                    //   { key: "student", width: 100, },
                    { key: "note", width: 160 },
                    { key: "actions", width: 180, }
                    //,sorter: (a, b) => a.complaindate > b.complaindate ? 1 : -1
                ]
                arr.map((e) => {
                    e.dataIndex = e.key
                    e.title = this.$t(e.key)
                    e.slots = { title: e.key + 'Title' }
                    e.scopedSlots = { customRender: e.slot ?? e.key }
                    return e
                })
                return arr;
            },
        },
        watch: {
            viewstatus() {
                console.log(this.viewstatus)
            },
            visible() {
                if (this.visible) {
                    this.tabledata = [];
                    this.fileList = [];
                }
            },
        },
        created() {

            this.loadcount = 1;
            reqPost('TRAINING', { prop: "getdept", dept: this.$props.dept }).then(res => {
                this.loadcount--;
                this.deptid = res.data[0].id;
                this.handle_get();
            })
            bus.$on("trainingclass", msg => {
                this.visible = msg._visible;
                if (msg.status == "ok") {
                    this.handle_get();
                }
            })
        },
        methods: {


            handle_get() {
                this.loadcount = 1;

                reqPost('TRAINING', { prop: "getclass", dept: this.deptid, name: this.values.name }).then(res => {
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
                this.dept = "";
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
                this.addi++;
                this.visible = true;
                this.actionstatus = "updateclass";
                this.params.classid = e.id;
                this.params.course = e.course;
                this.params.place = e.place;
                this.params.teacher = e.teacher;
                this.params.note = e.note;
                this.params.date_from = e.date_from;
                this.params.date_to = e.date_to;
                console.log(this.params)

            },

            handle_action() {
                if (this.actionstatus == 'upload') {
                    this.handle_saveupload();
                }
            },

            handle_add() {
                this.addi++;
                this.actionstatus = "add"
                this.visible = true;
                //  this.$router.push("../../../sys/data/training/class")
            },

            handle_delete(e) {
                var params = {
                    prop: "deleteclass",
                    course: e.course,
                    coursename: e.coursename,
                    place: e.place,
                    datefm: e.date_from,
                    dateto: e.date_to,
                    teacher: e.teacher,
                    note: e.note,
                    dept: this.deptid,
                    id: e.id
                }

                this.loadcount = 1;
                reqPost('TRAINING', params).then(res => {
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

            beforeUpload(file) {
                this.fileList = [file];
                this.setfile(file);
                //    return false;
            },

            handleRemove() {
                this.fileList.length = 0;
                this.tabledata.length = 0;
            },
            setfile(e) {
                var rr = [];
                this.tabledata.length = 0;
                rr = readExcel(e)
                this.scoreloadcount = 1
                setTimeout(() => {
                    this.tabledata = rr[0].sheet.map((e, i) => {
                        e.id = i
                        return e;
                    });
                    this.scoreloadcount--;
                }, 1000);  //1秒后执行代码 
            },
            handleChange(info) {
                info.fileList.map((e) => {
                    e.response && (this.fileid = e.response.id);
                })
            },
            handle_upload(e) {
                this.actionstatus = "upload";
                this.visible = true;
                this.coursename = e.coursename;
                this.course = e.id
            },

            handle_saveupload() {
                var temp = this.tabledata.map(e => {
                    return {
                        empno: e.empno,
                        score: e.score,
                        pass: e.pass,
                        class: this.course
                    }
                })
                var params = {
                    prop: "insertrecord",
                    rows: temp,
                    id: this.course,
                    files: this.fileid
                }
                reqPost('TRAINING', params).then(res => {
                    this.loadcount--;
                    if (res.data > 0) {
                        this.$message.success("Upload Success");
                        this.visible = false;
                        this.handle_get();
                    }
                    else {
                        this.$message.error("Upload Error");
                    }
                })
            },

            handle_look(e) {
                this.course = e.id;
                this.actionstatus = "look"
                this.visible = true;
                this.coursename = e.coursename;
                this.handle_searchscore();
            },
            handle_searchscore() {
                var params = {
                    prop: "lookscore",
                    class: this.course
                }
                this.scoreloadcount = 1;

                reqPost('TRAINING', params).then(res => {
                    this.scoreloadcount--;
                    this.tabledata = res.data.map(e => {
                        e.isaction = false;
                        return e;
                    });

                })
            },
            handle_deletescore(e) {
                var params = {
                    prop: "deletescore",
                    id: e.id
                }
                this.scoreloadcount = 1;
                reqPost('TRAINING', params).then(res => {
                    this.scoreloadcount--;
                    if (res.data > 0) {
                        this.handle_searchscore()
                    }
                    else {
                        this.$message.error("Delete Error");
                    }
                })
            },

            handle_updatescore(rec) {
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
                        this.actionstatus = "update"
                    }
                    return e;
                })

            },
            handle_saveupdatescore(e) {
                var params = {
                    prop: "updatescore",
                    id: e.id,
                    score: e.score,
                    pass: e.pass
                }
                this.scoreloadcount = 1;
                reqPost('TRAINING', params).then(res => {
                    this.scoreloadcount--;
                    if (res.data > 0) {
                        this.handle_searchscore()
                    }
                    else {
                        this.$message.error("Upadte Error");
                    }
                })
            },
            handle_close(rec, index) {
                if (this.actionstatus == "update") {
                    this.tabledata.splice(index, 1, this.temp_rec);
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

    .img {
        width: 2rem;
        height: 2rem;
        border-radius: .04rem;
        margin-left: .3rem;
        margin-top: .3rem;
        object-fit: cover;
    }
</style>