<template>
    <div>
        <a-modal :visible="visible" :title="title" :width="listWidth" :footer="null" @cancel="handleCancel">
            <a-spin :spinning="loading">
                <a-table class="basebg" :size="'small'" :bordered="true" :columns="cols" :pagination="false"
                    :data-source="tabledata" :loading="listLoading" rowKey='id' :scroll="scroll">
                    <template slot="no" slot-scope="text,record,index">
                        {{index+1}}
                    </template>
                    <template slot="fmDate" slot-scope="text, record">
                        <span v-if="!record.isaction">
                            {{text}}
                        </span>
                        <a-date-picker v-if="record.isaction" v-model="formvalues.fmDate" @change="fmdate_change" />

                    </template>
                    <template slot="toDate" slot-scope="text, record">
                        <span v-if="!record.isaction">
                            {{text}}
                        </span>
                        <a-date-picker v-if="record.isaction" v-model="formvalues.toDate" @change="todate_change" />
                    </template>
                    <template slot="weight" slot-scope="text, record">
                        <span v-if="!record.isaction">
                            {{text}}
                        </span>
                        <a-input v-if="record.isaction" v-model="formvalues.weight"
                            oninput="value=value.replace(/[^\d.]/g,'') "></a-input>
                    </template>

                    <template slot="actions" slot-scope="text, record">
                        <div v-if="!record.isaction">
                            <a-button :class="'warning'" size="small" @click="handle_edit(record)">
                                <a-icon type="edit" />
                            </a-button>
                        </div>

                        <div v-if="record.isaction">
                            <a-button :class="'success'" size="small" @click="handle_save(record)">
                                <a-icon type="save" />
                            </a-button>
                            &nbsp;&nbsp;&nbsp;
                            <a-button size="small" type="danger" @click="handle_close(record)">
                                <a-icon type="close" />
                            </a-button>
                        </div>
                    </template>

                </a-table>
            </a-spin>

        </a-modal>
    </div>
</template>

<script>
    import { reqPost } from '@/services/base'
    import bus from './bus.js'
    import { merge, moment } from '@/utils/util'
    import { mapState } from 'vuex'
    const DataSet = require('@antv/data-set')
    export default {
        name: 'iotDetail',
        props: {
            actionFix: String,
            paras: { type: Array, default: () => [] },
            title: { type: String, default: "List" },
            visible: { type: Boolean, default: false }
        },

        i18n: merge(require('@/i18n'), require('./i18n')),
        data() {
            return {
                loadcount: 0,
                listWidth:
                    document.documentElement.clientWidth * 0.8 > 2000
                        ? 2000
                        : document.documentElement.clientWidth * 0.8,
                listLoading: false,
                listData: [],

                isaction: false,
                tabledata: [],
                flag: false,
                formvalues: {
                    prop: "updateDist",
                    id: "",
                    fmDate: null,
                    toDate: null,
                    weight: 0
                },

            }
        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { y: this.pageMinHeight - 200 }
            },
            loading() {
                return this.loadcount > 0
            },
            tablewidth() {
                return this.cols.length * 300 + 20
            },
            cols() {
                var arr = [
                    { key: 'no', },
                    { key: 'deptname', },
                    { key: 'equipname', },
                    { key: 'fmDate', },
                    { key: 'toDate', },
                    { key: 'weight', },
                    { key: 'actions', },
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
            this.tabledata = this.paras.map(e => {
                e.isaction = false;
                return e;
            });
        },
        mounted() { },
        methods: {
            fmdate_change(date, dateString) {
                this.formvalues.fmDate = dateString;
            },
            todate_change(date, dateString) {
                this.formvalues.toDate = dateString;
            },
            handle_edit(rec) {
                var flag = 1;
                for (var i of this.tabledata) {
                    if (i.isaction) {
                        this.$message.warning("有編輯尚未保存")
                        flag = -1;
                    }
                }
                this.tabledata = this.tabledata.map(e => {
                    if (e.id == rec.id && flag == 1) {
                        e.isaction = true;
                        for (var key in e) {
                            if (['fmDate', 'toDate'].includes(key)) { 
                                this.formvalues[key] = (e[key] != null ? moment(e[key], 'YYYY-MM-DD') : null) || null
                            }
                            else {
                                this.formvalues[key] = e[key]
                            }
                        }
                        // this.formvalues.fmDate = e.fmDate;
                        // this.formvalues.toDate = e.toDate;
                        // this.formvalues.weight = e.weight;
                        // this.formvalues.id = e.id;
                    }
                    return e;
                })
            },

            handle_close(rec) {
                this.tabledata = this.tabledata.map(e => {
                    if (e.id == rec.id) {
                        e.isaction = false;
                    }
                    return e;
                })
            },
            handle_save(rec) {
                if (this.formvalues.fmDate > this.formvalues.toDate) {
                    this.$message.warning("起始日期不能大於結束日期")
                }
                else {
                    this.loadcount = 1;
                    reqPost("EF_IOT", this.formvalues).then(res => {
                        if (res.data > 0) {
                            this.loadcount--;
                            this.flag = true;
                            this.$message.success("Success")
                            var temp = []
                            temp = this.tabledata.map(e => {
                                // e.fmDate = moment(e.fmDate, 'YYYY/MM/DD') || null
                                // e.toDate = moment(e.toDate, 'YYYY/MM/DD') || null
                                if (e.id == rec.id) {
                                    for (var key in e) {
                                        if (['fmDate', 'toDate'].includes(key)) {
                                            //   e[key] = moment(this.formvalues[key], 'YYYY/MM/DD') || null
                                            e[key] = moment(this.formvalues[key]).format('YYYY-MM-DD') || null
                                        }
                                        else {
                                            e[key] = this.formvalues[key]
                                        }
                                    }
                                    e.isaction = false;
                                }
                                return e;
                            })
                            const ds = new DataSet()
                            const dv = ds
                                .createView()
                                .source(temp)
                                .transform({
                                    type: 'sort-by',
                                    fields: ['fmDate'], // 根据指定的字段集进行排序，与lodash的sortBy行为一致
                                    order: 'DESC', // 默认为 ASC，DESC 则为逆序
                                })
                            this.tabledata = dv.rows;

                        }
                        else {
                            this.flag = false;
                            this.$message.error("Error")
                        }
                    })
                }
            },
            handleCancel() {
                bus.$emit("clicktype", {
                    detail_visible: false,
                    status: this.flag ? "ok" : "false"
                })
            },
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
</style>