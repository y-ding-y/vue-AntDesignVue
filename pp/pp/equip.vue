<template>
    <ef-main :id="imageId" :isMainPage="true" :title="$t('pageTitle')" :loading="loading" :openSubmit="true"
        :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" @submit="submit" @resize="resize">

        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns"
            :pagination="{ pageSize }" :data-source="tableData" @change="onTableChange"
            :scroll="{ x: 1800, y: pageMinHeight - 200 }">
            <template slot="no" slot-scope="txt, rec ,index">{{ (currentPage - 1) * pageSize + index + 1 }}
            </template>

            <template slot="description" slot-scope="text, record">
                <a-tooltip :title="record.description">
                    <div style=" text-overflow: ellipsis;
                            overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                        {{ record.description }}
                    </div>
                </a-tooltip>
            </template>

            <template slot="attrcount" slot-scope="t, r"><a @click="onOpenList(r)"><b>{{ t }}</b></a></template>

        </a-table>
        <v-detaillist :vkey="key" :paras="paras" :prop="'attr'" :title="title" :action-fix="'cpdequipdetail'">
        </v-detaillist>

    </ef-main>
</template>

<script>

    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    import { merge } from '@/utils/util'
    import VDetaillist from './detaillist'

    const formItems = [
        {
            key: 'type',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsPp.TYPE',
        },
        {
            key: 'building',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsPp.GTKBUILDING',
        },
        {
            key: 'floor',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsPp.GTKFLOOR',
        },
        {
            key: 'status',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsPp.GTKSTATUS',
        },
    ]


    export default {
        name: 'cpdequipdetail',
        i18n: merge(require('@/i18n'), require('./i18n')),
        components: { EfMain, VDetaillist },
        data() {
            return {
                imageId: 'fpyimage' + new Date().format('hiu'),
                tableId: 'fpytable' + new Date().format('hiu'),
                loadcount: 0,
                sSize: 'middle',
                formItems,
                key: 0,
                sortOrder: '',
                footerRowKey: '',
                tableData: [],
                currentPage: 1,
                pageSize: 100,
                paras: {},
                visible: false,
                formValues: {},
            }
        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            title() {
                return this.$t('detailTitle')
            },
            loading() {
                return this.loadcount > 0
            },

            columns() {
                var arr = [
                    { key: 'no', fixed: 'left', align: 'center', width: 90 },
                    { key: 'name', fixed: 'left', align: 'left', width: 200 },
                    { key: 'code', align: 'left', },
                    { key: 'typename', align: 'left', width: 200 },
                    { key: 'floorname', align: 'left', },
                    { key: 'statusname', align: 'left', },
                    { key: 'description', align: 'left', },
                    { key: 'ip', align: 'left', },
                    { key: 'addr', align: 'right', },
                    { key: 'attrcount', align: 'right', },
                    { key: 'updatetime', align: 'left', width: 180 },
                    { key: 'createtime', align: 'left', width: 180 },
                ].map((e) => {
                    e.dataIndex = e.key
                    e.title = this.$t(e.key)
                    e.slots = { title: e.key + 'Title' }
                    e.scopedSlots = { customRender: e.slot ?? e.key }
                    e.sorter = e.key === 'no' ? false : (a, b) => this.sorter(a, b, e.key)
                    return e
                });
                return arr;
            },
        },
        watch: {},
        created() { },
        mounted() { },
        methods: {
            submit(values) {
                console.log(values);
                this.formValues = { ...values }
                this.handle_search(values)
            },

            handle_search(values) {
                this.loadcount = 1;
                reqPost("SFCS_PP", { ...values, prop: "equip", orgId: "476" })
                    .then(res => {
                        this.loadcount--;
                        this.tableData = res.data;
                        console.log(res.data);
                    })
            },

            resize(size) {
                this.sSize = size
            },
            onTableChange(pagination, filters, sorter) {
                this.sortOrder = sorter.order
                this.currentPage = pagination.current
            },


            sorter(a, b, rcs) {
                if (rcs === 'addr' || rcs === 'attrcount') {
                    const aI = a[rcs] ?? 0
                    const bI = b[rcs] ?? 0
                    return Math.sign(aI - bI)
                } else {
                    const aN = a[rcs] || ""
                    const bN = b[rcs] || ""
                    return aN === bN ? 0 : aN > bN ? 1 : -1
                }
            },
            onOpenList(record) {
                this.key++
                this.visible = true
                const paras = {
                    ...this.formValues,
                    equip: record.key
                }
                this.paras = paras
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
</style>