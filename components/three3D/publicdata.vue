<template>
    <div>
        <a-button :class="'success'" style="width: 100%;" @click="handle_addapi">新增数据</a-button>
        <a-form-model layout="horizontal" :label-col=" { span: 4 }" :wrapper-col=" { span: 20 }">

            <a-form-model-item label="类型">
                <a-select v-model="datatype">
                    <a-select-option value="text" title="本地txt"> 本地txt</a-select-option>
                    <a-select-option value="excel" title="本地excel"> 本地excel</a-select-option>
                    <a-select-option value="phpapi" title=" php api"> php api</a-select-option>
                    <a-select-option value="nodejsemit" title="nodejs 推送"> nodejs emit</a-select-option>
                    <a-select-option value="nodejson" title="nodejs 返回"> nodejs on</a-select-option>
                    <a-select-option value="localforage" title="localForage"> localForage</a-select-option>
                </a-select>
            </a-form-model-item>
            <a-form-model-item label="名称">
                <a-input v-model="dataname" />
            </a-form-model-item>
            <a-form-model-item label="API">
                <a-input type='textarea' v-model="dataapi" />
            </a-form-model-item>
        </a-form-model>

        <a-tabs tab-position="left" v-model="actived" @change="change_tab">
            <a-tab-pane v-for="(item,index) in tablist" :key="index" :tab="item.dataname">
                <a-table :scroll="scroll" rowKey="id" :columns="item.tablecolumn" :data-source="item.tabledata"
                    :pagination="false">

                </a-table>
            </a-tab-pane>

        </a-tabs>

    </div>
</template>

<script>
    import { merge } from '@/utils/util' // saveToExcel  

    const i18n = require('./i18n')
    export default {
        i18n: merge(require('@/i18n'), i18n),
        components: {},
        props: {
            ss: {
                type: Object, default: () => ({})
            },
            keyid: { type: Number, default: 0, },
            tabs: { type: Array, default: () => [] }
        },

        data() {
            return {
                i18n,
                loadcount: 0,
                tablist: [],
                actived: 0,

                datatype: "phpapi",
                dataname: "公共数据",
                dataapi: "",
            }
        },
        computed: {
            loading() {
                return this.loadcount > 0
            },
            scroll() {
                return { x: 500, y: this.pageMinHeight - 132 }
            },
        },

        created() {
            this.tablist = this.tabs 
            this.$localForage.setItem("3dpublicdata", this.tabs)
        },
        mounted() { },
        methods: {
            handle_addapi() {
                this.tablist.push({
                    key: this.tablist.length,
                    datatype: this.datatype,
                    dataapi: this.dataapi,
                    dataname: this.dataname,
                });
                this.$localForage.setItem("3dpublicdata", this.tablist)
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
</style>n
