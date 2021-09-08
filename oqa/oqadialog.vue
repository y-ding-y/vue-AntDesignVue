<template>
    <div>
        <a-modal :visible="visible" :width="listWidth" @cancel="handleCancel" :maskClosable="false" :footer="null">
            <upload-form v-if="visible" :isMainPage="false" :form-items="formItems" @submit="submit" :loading="loading">

            </upload-form>
        </a-modal>
    </div>


</template>
<script>
    import { reqPost } from '@/services/base'
    import UploadForm from '@/components/page/uploadform'
    import { merge } from '@/utils/util'
    import bus from './bus.js'
    //厂别，投诉日期，投诉类别，投诉人，机种，问题描述，原因类别，上传文件

    const i18n = require('./i18n')
    export default {
        name: 'oqadialog',
        props: {
            actionFix: String,
            complain: { type: Object, default: () => ({}) },
            title: { type: String, default: "List" },
            visible: { type: Boolean, default: false }
        },
        components: { UploadForm, },
        i18n: merge(require('./i18n'), i18n),
        data() {
            return {
                loadcount: 0,
                listTableWidth: 2800,
                modellist: [],
                listWidth:
                    document.documentElement.clientWidth * 0.8 > 2000
                        ? 2000
                        : document.documentElement.clientWidth * 0.8,

            };
        },
        computed: {
            loading() {
                return this.loadcount > 0;
            },
            formItems() {
                var arr = [];
                var filelist = this.complain.files.split(',').map((e, index) => {
                    return {
                        uid: index,
                        name: (this.complain.files_name && this.complain.files_name.split('*')[index]) || "",
                        status: 'done',
                        url: process.env.VUE_APP_API_WEB_URL + '/file/' + e,
                        id: e
                    }
                })
                arr = [

                    { key: 'plant', type: 'select', paraKey: 'para.121', allowClear: false, value: this.complain.plant, span: "1" },
                    {
                        key: 'complaindate', type: 'date', expandDayTime: true, format: 'YYYY-MM-DD',
                        maxDays: 366, value: this.complain.complaindate, allowClear: false, span: "1"
                    },
                    { key: 'model', type: 'select', paraKey: 'SfcsRSBU.FAMILY', allowClear: false, value: this.complain.model, span: "1" },
                    {
                        key: 'customer', type: 'select', paraKey: 'para.131', allowClear: false, value: this.complain.customer, span: "3", style: 'width:250px;max-width:600px',
                    },
                    { key: 'category', type: 'radio', allowClear: false, paraKey: 'para.123', value: this.complain.category, span: "3" },
                    // { key: 'category', type: 'select', allowClear: false, paraKey: 'para.123', value: "125", span: "2" },
                    { key: 'cause', type: 'select', allowClear: false, paraKey: 'para.124', value: this.complain.cause, span: "3" },
                    { key: 'description', type: "textarea", span: "3", value: this.complain.description },
                    { key: 'files', type: 'uploadfile', value: filelist, api: process.env.VUE_APP_API_BASE_URL + "/file/upload", span: "3" },



                    // { key: 'plant', type: 'select', paraKey: 'SfcsSmt.BU', allowClear: false, value: this.complain.plant },
                    // {
                    //     key: 'complaindate', type: 'date', expandDayTime: true, format: 'YYYY-MM-DD',
                    //     maxDays: 366, value: this.complain.complaindate, allowClear: false,
                    // },
                    // { key: 'category', type: 'select', allowClear: false, paraKey: 'SfcsSmt.LINE', value: this.complain.category },
                    // { key: 'customer', type: 'select', paraKey: 'SfcsSmt.BU', allowClear: false, value: this.complain.customer },
                    // { key: 'model', type: 'select', paraKey: 'SfcsSmt.CLASS', allowClear: false, value: this.complain.model },
                    // { key: 'cause', type: 'select', allowClear: false, paraKey: 'SfcsSmt.LINE', value: this.complain.cause },
                    // { key: 'description', type: "textarea", value: this.complain.description },
                    // { key: 'files', type: 'uploadfile', value: filelist },
                ]
                return arr;
            },
        },
        created() {
            this.loadcount = 1;
            reqPost('PARA_SET', { paras: [{ "key": "SfcsRSBU.FAMILY" }] }).then(res => {
                this.loadcount--;
                this.modellist = res.data;
            })
        },
        methods: {
            submit(values) {
                this.modellist[0].data.some(e => {
                    if (e.key == values.model) {
                        values.model = e.name;
                        return true;
                    }
                })
                var fileuid = "";
                var filename = "";
                values.files && values.files.map((e, index) => {
                    if (e.response) {
                        if (index != values.files.length - 1) {
                            fileuid = fileuid + e.response.id + ",";
                            filename = filename + e.response.name + "*";
                        }
                        else {
                            fileuid = fileuid + e.response.id;
                            filename = filename + e.response.name;
                        }
                    }
                    else {
                        if (index != values.files.length - 1) {
                            fileuid = fileuid + e.id + ",";
                            filename = filename + e.name + "*";
                        }
                        else {
                            fileuid = fileuid + e.id;
                            filename = filename + e.name;
                        }
                    }
                })
                values.fileuid = fileuid;
                values.filename = filename;
                values.id = this.complain.id;
                values.prop = "update";
                if (values.fileuid == "") {
                    this.$message.error("請選擇附件上傳")
                }
                else {
                    this.loadcount = 1;
                    this.ifrefresh = false,
                        reqPost("FILE_ACTION", values).then(res => {
                            if (res.data == "1") {
                                this.$message.success("Upload Success")
                                this.loadcount--;
                                this.i++;
                                this.ifrefresh = true; 
                                bus.$emit("clicktype", {
                                    _visible: false, updateaction: "ok"
                                })
                            }
                            else {
                                this.$message.error("Upload Error")
                            }
                        })
                }

            },

            handleCancel() {
                bus.$emit("clicktype", {
                    _visible: false, updateaction: "fail"
                })
            },
        },
    };
</script>