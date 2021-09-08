<template>
    <div>

        <a-modal :visible="visible" :width="listWidth" @cancel="handleCancel" :maskClosable="false" :footer="null">
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
                                <img :src="icons+'/file/'+item+'/small' " :large="icons+'/file/'+item+'/large'"
                                    class="img" :preview="item">
                            </transition>

                            <a v-else-if="rec.files_name"
                                :href="icons+'/file/'+item ">{{rec.files_name.split('*')[index]}}
                            </a>
                            <span v-else>{{item}} </span>
                        </div>
                    </div>
                </template>
            </a-table>
        </a-modal>

    </div>


</template>
<script>
    import { reqPost } from '@/services/base'
    import { merge } from '@/utils/util'
    import bus from './bus.js'
    import { mapState } from 'vuex'

    const i18n = require('./i18n')
    export default {
        name: 'oqadetail',
        props: {
            actionFix: String,
            complain: { type: Object, default: () => ({}) },
            title: { type: String, default: "List" },
            visible: { type: Boolean, default: false },
            paras: { type: Object, default: () => ({}) },
        },
        i18n: merge(require('./i18n'), i18n),

        data() {
            return {
                loadcount: 0,
                sSize: 'middle',
                listTableWidth: 2800,
                listWidth:
                    document.documentElement.clientWidth * 0.8 > 2000
                        ? 2000
                        : document.documentElement.clientWidth * 0.8,
                tableData: [],

            };
        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            loading() {
                return this.loadcount > 0;
            },
            icons() {
                return process.env.VUE_APP_API_WEB_URL;
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
        created() {
            this.loadcount = 1;
            this.handle_search(this.paras)
        },
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
                values.prop = "chart";
                reqPost("FILE_ACTION", values).
                    then(res => {
                        this.loadcount--;
                        // this.tableData = this.formatFile(res.data,'files');
                        this.tableData = res.data;
                    })
            },
            handleCancel() {
                bus.$emit("clicktype", {
                    _visible: false, updateaction: "fail"
                })
            },
        },
    };
</script>