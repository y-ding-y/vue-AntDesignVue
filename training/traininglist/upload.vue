<template>
    <div>
        <ef-main :isMainPage="false" :title="$t('upload')" :loading="loading" @resize="resize">
            <a-steps :current="active" finish-status="finish" align-center key=key1>
                <a-step>
                    <template slot="description">
                        <br>
                        <b>請選擇：</b>
                        <a-select v-if="configlist" v-model="configid" style="width:100px">
                            <a-select-option v-for="(item) in configlist" :value="item.id" :key="item.id">
                                {{item.name}}
                            </a-select-option>
                        </a-select>
                    </template>
                </a-step>
                <a-step>
                    <template slot="description">
                        <br>
                        <a-upload :file-list="fileList" :remove="handleRemove" :before-upload="beforeUpload"
                            @change="setfile">
                            <a-button type="primary">
                                <a-icon type="upload" /> Select File
                            </a-button>
                        </a-upload>
                    </template>
                </a-step>
                <a-step>
                    <template slot="description">
                        <br>
                        <a-button type="primary" @click="uploadexcel">Click Upload <i class="a-icon-upload"></i>
                        </a-button>
                    </template>
                </a-step>
                <a-step>
                    <template slot="description">
                        <br>
                        Success
                    </template>
                </a-step>
            </a-steps>
            <br>
            <a-table class="basebg" :scroll="{  y: tableheight }" :bordered="true" :columns="columns"
                :pagination="false" :data-source="tabledata" rowKey="NO" :size="sSize">
                <template slot="NO" slot-scope="txt, rec,i">
                    {{i+1}}
                </template>
                <template slot="description" slot-scope="text, record">
                    <a-tooltip :title="record.description">
                        <div
                            style="text-overflow: ellipsis;overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                            {{ record.description }}
                        </div>
                    </a-tooltip>
                </template>
            </a-table>
        </ef-main>

    </div>
</template>

<script>
    import { reqPost } from '@/services/base'
    import { readExcel } from './upload.js'
    import moment from 'moment'
    import EfMain from '@/components/page/main'
    import { merge } from '@/utils/util'
    import { mapState } from 'vuex'

    const i18n = require('./i18n')
    export default {
        name: 'efetd',
        i18n: merge(require('@/i18n'), i18n),
        data() {
            return {
                fileList: [],
                active: 0,
                filedata: "",
                tabledata: [],
                total: 0,
                pagesize: 10,
                currentPage: 1,
                key1: 1,
                selectedRowKeys: [],
                sSize: 'small',
                loadcount: 0,
                configlist: [],
                configid: "",
            };
        },
        created() {
            reqPost('APS', { prop: "getapsconfig" }).then(res => {
                this.loadcount--;
                if (res.data.length > 0) {
                    //  this.plantlist = res.data[0].data;
                    this.configlist = res.data.map(e => {
                        return {
                            id: String(e.id),
                            name: e.name
                        }
                    });
                    this.configid = this.configlist[0].id
                }
                else {
                    this.$message.error("請先維護部門")
                }
            })
        },
        components: { EfMain },
        computed: {

            loading() {
                return this.loadcount > 0
            },
            ...mapState('setting', ['pageMinHeight']),
            tableheight() {
                return this.pageMinHeight - 330;
            },
            columns() {
                var temp = [{ key: "NO", width: 80 },
                { key: "pn", width: 150 },
                { key: "description", },
                { key: "cn_description", },];
                return temp.map(e => {
                    e.dataIndex = e.key
                    e.title = e.dataIndex
                    e.slots = { customRender: e.key }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e;
                })
            },
        },
        watch: {
            configid() {
                if (this.configid != "") {
                    this.active++;
                }
            },
        },
        methods: {

            beforeUpload(file) {
                this.fileList = [file];
                return false;
            },
            getIndex($index) {  //表格序号 
                return ((this.currentPage - 1) * this.pagesize + $index + 1)
            },
            resize(size) {
                this.sSize = size
            },
            dateFormat(date) {
                return moment(date).format('YYYY-MM-DD')
            },
            clear() {
                ++this.key1;
                this.handleRemove();
            },
            handleRemove() {
                this.fileList.length = 0;
                this.tabledata.length = 0;
                this.active = 0;
            },
            uploadexcel() {
                var temp = this.tabledata.map(e => {
                    return {
                        PN: e.pn,
                        DESCRIPTION: e.description,
                        CN_DESCRIPTION: e.cn_description,
                        config: this.configid
                    }
                })

                var params = {
                    prop: 'insertcom',
                    rows: temp,
                }
                console.log(params)
                this.loadcount = 1
                reqPost('APS', params).then((res) => {
                    this.loadcount--
                    this.active = this.active + 1;
                    if (res.data) {
                        this.$message.success('Success')
                        this.clear()
                    } else {
                        this.$message.error('Fail')
                    }
                })
            },
            setfile(e) {
                this.filedata = e.file;
                var rr = [];
                this.tabledata.length = 0;
                this.total = 0;
                rr = readExcel(e.file)

                this.loadcount = 1
                setTimeout(() => {
                    this.tabledata = rr[0].sheet.map((e, i) => {
                        e.NO = i
                        return e;
                    });
                    this.loadcount--;
                }, 1000);  //1秒后执行代码


                this.active = this.active + 1;
            },
        }
    }
</script>

<style>
    .a-upload {
        width: 120px;
        height: 35px
    }
</style>