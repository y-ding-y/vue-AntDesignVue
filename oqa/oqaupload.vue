<template>
    <upload-form :isMainPage="true" :form-items="formitem" @submit="submit" :loading="loading" :key="i">

    </upload-form>

</template>
<script>
    import { reqPost } from '@/services/base'
    import UploadForm from '@/components/page/uploadform'
    import { merge } from '@/utils/util'

    //厂别，投诉日期，投诉类别，投诉人，机种，问题描述，原因类别，上传文件
    const formItems = [
        { key: 'plant', type: 'select', paraKey: 'para.121', allowClear: false, value: "122", span: "1" },
        {
            key: 'complaindate', type: 'date', expandDayTime: true, format: 'YYYY-MM-DD',
            maxDays: 366, value: new Date().format('Y-m-d'), allowClear: false, span: "1"
        },
        { key: 'model', type: 'select', paraKey: 'SfcsRSBU.FAMILY', allowClear: false, value: "369", span: "1" },
        {
            key: 'customer', type: 'select', paraKey: 'para.131', allowClear: false, value: "132", span: "3", style: 'width:250px;max-width:600px',
        },
        { key: 'category', type: 'radio', allowClear: false, paraKey: 'para.123', value: "125", span: "3" },
        // { key: 'category', type: 'select', allowClear: false, paraKey: 'para.123', value: "125", span: "2" },
        { key: 'cause', type: 'select', allowClear: false, paraKey: 'para.124', value: "129", span: "3" },
        { key: 'description', type: "textarea", span: "3" },
        { key: 'files', type: 'uploadfile', picture: true, api: process.env.VUE_APP_API_BASE_URL + "/file/upload", span: "3" },
    ]
    const i18n = require('./i18n')
    export default {
        name: 'oqaupload',
        components: { UploadForm },
        i18n: merge(require('./i18n'), i18n),
        data() {
            return {
                loadcount: 0,
                ifrefresh: true,
                i: 0,
                modellist: [],
            };
        },
        computed: {
            loading() {
                return this.loadcount > 0;
            },
            formitem() {
                var res = JSON.parse(JSON.stringify(formItems));
                if (this.ifrefresh) { 
                    res = JSON.parse(JSON.stringify(formItems));
                }
                return res;
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
                    if (index != values.files.length - 1) {
                        fileuid = fileuid + e.response.id + ",";
                        filename = filename + e.response.name + "*";
                    }
                    else {
                        fileuid = fileuid + e.response.id;
                        filename = filename + e.response.name;
                    }
                })
                values.fileuid = fileuid;
                values.filename = filename;
                values.prop = "insertComplain";

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
                            }
                            else {
                                this.$message.error("Upload ERROR")
                            }
                        })
                }

            },

        },
    };
</script>