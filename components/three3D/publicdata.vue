<template>
    <div>
        <a-tabs tab-position="top">
            <a-tab-pane key="11" tab="Public Data">
                <a-tabs tab-position="left" v-model="actived" @change="change_tab">
                    <a-tab-pane v-for="(item,index) in tablist" :key="index" :tab="item.name">
                        <a-table :scroll="scroll" rowKey="id" :columns="item.column" :data-source="item.tabledata"
                            :pagination="false">

                            <template v-for="(col,index) in item.column" :slot="col.key" slot-scope="txt">

                                <a-tooltip :title="txt" :key="col.key+'-'+index">
                                    <div style=" text-overflow: ellipsis;
                                    overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                                        {{txt}}
                                    </div>
                                </a-tooltip>
                            </template>

                            <template slot="no" slot-scope="txt,rec,index">
                                {{index+1}}
                            </template> 

                        </a-table>

                    </a-tab-pane>
                </a-tabs>
            </a-tab-pane>

            <a-tab-pane key="12" tab="Form">
                <v-form title="Public Data" :form-items="formItems" @submit="submit" :rules="rules" :form="form">
                </v-form>
            </a-tab-pane>
        </a-tabs>
    </div>
</template>

<script>
    import { merge } from '@/utils/util' // saveToExcel  
    import VForm from '@/components/page/drawform'
    import { mapState } from 'vuex'
    // import bus from './bus.js'

    const i18n = require('./i18n')
    const rules = {
        id: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
        dataparam: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],

    }

    const form = {
        id: "",
        dataparam: "{'name':'',datasource:'','api':{'':'','':''}}",
    }

    export default {
        i18n: merge(require('@/i18n'), i18n),
        components: {
            VForm
        },
        props: {
            ss: {
                type: Object, default: () => ({})
            },
            keyid: { type: Number, default: 0, },
            tabs: { type: Array, default: () => [] }
        },

        data() {
            return {
                i18n, rules, form,
                loadcount: 0,
                //  tablist: [],
                actived: 0,

                datatype: "phpapi",
                dataname: "公共数据",
                dataapi: "",
                dataparam: "{'name':'',datasource:'','api':{'':'','':''}}",


            }
        },
        computed: {
            tablist() {
                console.log(this.$store.getters['tdpublic/commondata'])
                return this.$store.getters['tdpublic/commondata']
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),
            loading() {
                return this.loadcount > 0
            },
            scroll() {
                return { x: 50, y: this.pageMinHeight - 180 }
            },
            formItems() {
                var arr = [
                    { key: 'id', type: 'select', paraKey: 'PARA.528' },
                    { key: 'dataparam', type: 'textarea', },

                ]
                return arr;
            },
            drawerheight() {
                return this.pageMinHeight - 30
            },
        },
        watch: {
        },
        created() {
            // this.tablist = this.tabs
            // bus.$on("publicdata", msg => {
            //     console.log(msg)
            //     this.tablist = msg
            // })

            //bus.$emit("close", this.tabs)
            // this.$funAxios.reqPost("PARA_SET", { paras: [{ key: "PARA.528" }] }).then(res => {
            //     console.log(res.data)
            //     this.tablist = res.data[0].data.map(e => {
            //         let temp = eval("(" + e.note + ")")
            //         console.log(temp)
            //         if (e.note != null)
            //             this.$funAxios.reqPost(temp.api, temp.params).then(res2 => {
            //                 console.log(res2.data)
            //                 e.tabledata = res2.data
            //                 e.column = [{
            //                     dataIndex: "no", key: "no", title: "NO", width: 80,
            //                     scopedSlots: { customRender: "no" }
            //                 }]
            //                 for (var key in res2.data[0]) {
            //                     e.column.push({
            //                         dataIndex: key,
            //                         key: key,
            //                         title: key,
            //                         width: 100,
            //                     })
            //                 }
            //             })
            //         return e
            //     })

            // })
        },
        mounted() { },
        methods: {
            submit(value) {
                console.log(value)
                let paras = eval("(" + value.dataparam + ")")
                console.log(paras)
                value.prop = "updateef2para"
                value.note = value.dataparam
                this.$funAxios.reqPost("APIDATE", value).then(res => {
                    if (res.data > 0) {
                        this.$message.success("Upload Success")
                        this.$funAxios.reqPost(paras.api, paras.params).then(res => {
                            console.log(res.data)
                        })
                    }
                })

            },
            change_tab(e) {
                console.log(e)
                this.datatype = this.tablist[e].datatype
                this.dataapi = this.tablist[e].dataapi
                this.dataname = this.tablist[e].dataname
            },
        },
    }
</script>

<style lang="less" scoped>
    .success,
    .success:hover,
    .success:visited {
        background-color: @success-color;
        color: white;
    }

    .cancel,
    .cancel:hover,
    .cancel:visited {
        background-color: @primary-2;
        color: white;
    }
</style>
