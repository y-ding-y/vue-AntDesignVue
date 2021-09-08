<template>
    <ef-main :isMainPage="true" title="" :loading="loading" :form-items="formItems" :openSubmit="true" @submit="submit"
        @resize="resize">

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
                            v-if="rec.files_name && ['jpg','jpeg','png','pdf'].includes((rec.files_name.split('*')[index]).split('.')[1])">
                            <img :src="icons+'/file/'+item+'/small' " :large="icons+'/file/'+item+'/large'" class="img"
                                :preview="item">
                        </transition>

                        <a v-else-if="rec.files_name" :href="icons+'/file/'+item ">{{rec.files_name.split('*')[index]}}
                        </a>
                        <span v-else>{{item}} </span>
                    </div>
                </div>
            </template>

            <template slot="actions" slot-scope="text,rec">
                <a-button :class="'warning'" size="small" type="link" @click="handle_edit(rec)">
                    <a-icon type="edit" />
                </a-button>

                <a-popconfirm title="Are you sure delete ?" ok-text="Yes" cancel-text="No"
                    @confirm="handle_delete(rec)">
                    <a-button :class="'danger'" size="small" type="link">
                        <a-icon type="delete" />
                    </a-button>
                </a-popconfirm>



            </template>
        </a-table>
        <v-dialog v-if="visible" :key='i' :visible="visible" :complain="complain" :title="'Detail'"
            :action-fix="'oqatable'"></v-dialog>

    </ef-main>
</template>

<script>

    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import { merge } from '@/utils/util'
    import VDialog from './oqadialog.vue'
    import bus from './bus.js'
    import { mapState } from 'vuex'


    // const formItems = [
    //     // {
    //     //     key: 'month',
    //     //     dataIndex: 'Time',
    //     //     type: 'month',
    //     //     format: 'YYYY-MM',
    //     //     allowClear: false,
    //     //     value: new Date().format('Y-m-d'),
    //     // },
    //     {
    //         key: 'date',
    //         dataIndex: 'Time',
    //         type: 'range',
    //         format: 'YYYY-MM-DD',
    //         maxDays: 366,
    //         value: [new Date(year, 0, 1), new Date(year, 11, 31)],
    //         allowClear: false,
    //     },
    // ]

    export default {
        name: 'oqatable',
        i18n: merge(require('@/i18n'), require('./i18n')),
        components: { EfMain, VDialog },
        data() {
            return {
                imageId: 'fpyimage' + new Date().format('hiu'),
                tableId: 'fpytable' + new Date().format('hiu'),
                loadcount: 0,
                sSize: 'middle',
                sortOrder: '',
                footerRowKey: '',
                tableData: [],
                paras: {},
                visible: false,
                i: 0,
                complain: {},
                s: "",
            }
        },
        computed: {
            formItems() {
                var year = new Date().format('Y')
                return [
                    {
                        key: 'date',
                        dataIndex: 'Time',
                        type: 'range',
                        format: 'YYYY-MM-DD',
                        maxDays: 366,
                        value: [new Date(year, 0, 1), new Date(year, 11, 31)],
                        allowClear: false,
                    },
                ]
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),
            icons() {
                return process.env.VUE_APP_API_WEB_URL;
            },
            loading() {
                return this.loadcount > 0
            },
            scroll() {
                return { x: 1900, y: this.pageMinHeight - 220 }
            },
            columns() {
                var arr = [
                    { key: 'no', width: 90, align: "center" },
                    { key: "complaindate", width: 150 },
                    { key: "customer_name", width: 130 },
                    { key: "model" },
                    { key: "category_name" },
                    { key: "plant_name" },
                    { key: "cause_name" },
                    // { key: "created_by" },
                    { key: "description" },
                    { key: "files" },
                    { key: "created_at", width: 180 },
                    { key: "actions", width: 100 }
                ].map((e) => {
                    e.dataIndex = e.key
                    e.title = this.$t(e.key)
                    e.slots = { title: e.key + 'Title' }
                    e.scopedSlots = { customRender: e.slot ?? e.key }
                    e.sorter = e.key === 'no' ? false : (a, b) => this.sorter(a, b, e.key)
                    return e
                })
                return arr;
            },
        },
        watch: {},
        created() {
            bus.$on("clicktype", msg => {
                this.visible = msg._visible;
                if (msg.updateaction == "ok") {
                    this.submit(this.paras)
                }
            })
        },
        mounted() { },
        methods: {

            submit(values) {
                values.idate = [];
                values.idate[0] = values.Time[0].substr(0, 7);
                values.idate[1] = values.Time[1].substr(0, 7);
                this.paras = { ...values };
                this.handle_search(values)
            },
            handle_search(values) {
                this.loadcount = 1;
                values.prop = "getoqa";
                reqPost("FILE_ACTION", values).
                    then(res => {
                        this.loadcount--;
                        // this.tableData = this.formatFile(res.data,'files');
                        this.tableData = res.data;
                    })
            },

            formatFile(data, col) {
                const id = data.map(e => e[col].split(',')).flat(2)
                let files = null;
                reqPost("", { id }).then(res => {
                    files = res.data.map(e => ({ [e.id]: e }))
                })
                data.foreach(e => {
                    e.files = e.files.split(',').map(f => {
                        return files[f]
                    })
                })
                return data
            },
            // file_icon(file){
            //     const {file_kind,file_type,file_path} =file
            //     if(file_kind.startwith('image')){
            //         return env('VUE_APP_API_WEB_URL')+'/file/'+file_path
            //     }else if(['jpg','gif','doc','docx'].includes(file_type)) {
            //         return 'image/icon/'+file_type+'.gif'
            //     }else{
            //         return 'image/icon/default.icon.gif'
            //     }
            // },

            resize(size) {
                this.sSize = size
            },

            handle_delete(element) {
                this.loadcount = 1;
                reqPost("FILE_ACTION", {
                    prop: "delete",
                    id: element.id,
                }).
                    then(res => {
                        this.loadcount--;
                        if (res.data > 0) {
                            this.$message.success("Delete Success")
                            this.handle_search(this.paras);
                        }
                        else {
                            this.$message.error("Delete Error")
                        }
                    })
            },

            handle_edit(element) {
                this.i++;
                this.visible = true;
                this.complain = element;
            },

            // handle_show(element) { 
            //     reqShow("FILE", { id: element }).then(res => {  
            //         return ' <img :src="data:image/jpg;base64, style="width: 100px;height: 100px;">';

            //         //     var str = "";
            //         //     if (['jpg', 'jpeg', 'pdf', 'png'].includes(res.headers.filetype)) { 
            //         //         this.s=res.data;
            //         //         str ="<img src='data:image/jpg;base64,"+res.data+"'>" ;
            //         //     }
            //         //     else { 
            //         //         str = "<a>" + res.headers.filename + "</a>"
            //         //     }
            //         // console.log(str); 
            //         // return str;
            //     })

            // },
        },
    }
</script>

<style scoped lang="less">
    .title {
        font-weight: bold;
    }

    .basebg {
        background-color: @base-bg-color;
    }

    .danger {
        color: @error-color;
    }

    .warning {
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