<template>
    <div>
        <a-table class="basebg" :id="id" :size="size" :bordered="true" :columns="columns" :pagination="false"
            :data-source="tabledata" :rowClassName="rowClassName" @change="onTableChange">
            <template slot="no" slot-scope="txt, rec, index">{{
                getRowNo(index)
                }}</template>
            <template slot="rate" slot-scope="txt">{{ (txt * 100).toFixed(2) + '%' }}</template>

            <template slot="sample" slot-scope="txt, rec">
                <span v-if="rec.key==''">{{txt}}</span>
                <span v-else-if="txt!=0">{{txt}}</span>
            </template>
            <template slot="fail" slot-scope="txt, rec"> 
                <a v-if="txt!=0" @click="handle_click(rec)">{{txt}}</a>

            </template>
            <template slot="miss" slot-scope="txt, rec">
                <span v-if="rec.key==''">{{txt}}</span>
                <span v-else-if="txt!=0">{{txt}}</span>
            </template>
            <template slot="dppm" slot-scope="txt, rec">
                <span v-if="rec.key==''">{{txt}}</span>
                <span v-else-if="txt!=0">{{txt}}</span>
            </template>
            <template slot="dppm_miss" slot-scope="txt,rec">
                <span v-if="rec.key==''">{{txt}}</span>
                <b v-else-if="txt!=0">{{txt}}</b>
            </template>

        </a-table>

        <defect-list :vkey="key" :paras="paras" :prop="'defect'" :title="title" :action-fix="actionFix"></defect-list>
    </div>
</template>

<script>
    import { merge } from '@/utils/util'
    import DefectList from '@/pages/sfcs/sys/modal/defectList'

    export default {
        name:"dppm_table",
        i18n: merge(require('@/i18n'), require('./i18n')),
        props: {
            actionFix:String,
            groupkey: String,
            size: String,
            id: String,
            data: Array,
            tabledata: Array,
            values: Object,
        },
        data() {
            return {
                sortOrder: '',
                sSize: 'middle',
                footerRowKey: '',
                itemkey: '',
                vshow: false,
                rc: '',
                rcname: '',

                visible: false,
                key: 0,
                paras: {},
            }
        },
        components: {
            DefectList,
        },
        computed: {
            columns() {
                return [
                    { title: this.$t('no'), dataIndex: 'no', key: 'no', align: 'center' },
                    { title: this.$t(this.groupkey), key: 'name', align: 'center', dataIndex: 'name', },
                    { title: this.$t('sample'), dataIndex: 'sample', key: 'sample', align: 'right', },
                    { title: this.$t('fail'), dataIndex: 'fail', key: 'fail', align: 'right', },
                    { title: this.$t('miss'), dataIndex: 'miss', key: 'miss', align: 'right', },
                    { title: this.$t('dppm'), dataIndex: 'dppm', key: 'dppm', align: 'right', },
                    { title: this.$t('dppm_miss'), dataIndex: 'dppm_miss', key: 'dppm_miss', align: 'right', },
                    { title: this.$t('rate'), dataIndex: 'rate', key: 'rate', align: 'right', },
                ].map((e) => {
                    e.slots = { title: e.key, customRender: e.key };

                    (e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }),
                        (e.sorter = e.key === 'no' ? false : (a, b) => this.sorter(a, b, e.key))
                    return e
                })
            },
            title() {
                return this.$t('title')
            },
        },
        methods: {
            rowClassName(record) {
                return record.key === this.footerRowKey
                    ? 'ef-sfcs-stat-table-footer'
                    : null
            },

            getRowNo(index) {
                const len = (this.tabledata ?? []).length
                return index === len - 1 ? '-' : index + 1
            },

            onTableChange(pagination, filters, sorter) {
                this.sortOrder = sorter.order
            },
            sorter(a, b, rcs) {
                const order = this.sortOrder
                const fKey = this.footerRowKey
                if (a.key === fKey) {
                    return order === 'ascend' ? 1 : -1
                } else if (b.key === fKey) {
                    return order !== 'ascend' ? 1 : -1
                } else if (rcs === 'name') {
                    const aN = a[rcs]
                    const bN = b[rcs]
                    return aN === bN ? 0 : aN > bN ? 1 : -1
                } else {
                    const aI = a[rcs] ?? 1
                    const bI = b[rcs] ?? 1
                    return Math.sign(aI - bI)
                }
            },
            handle_click(e) {
                this.key++
                this.visible = true;
                const paras = {
                    ...this.values,
                    ...{ group: '',  routeCode: 'OQA' },
                }
                paras[this.groupkey] = e.key
                this.listLoading = true
                this.paras = paras
                // let routeData = this.$router.resolve({ path: '/test' });
                // window.open(routeData.href, '_blank'); 
            },
        },
    }
</script>

<style lang="less">
    .ef-sfcs-stat-table-footer {
        font-weight: bold;
        background-color: @table-selected-row-bg;
    }
</style>

<style scoped lang="less">
    .basebg {
        background-color: @base-bg-color;
    }
</style>