<template>
    <ef-main :id="imageId" :isMainPage="true" :title="$t('yieldTitle')" :loading="loading" :openSubmit="true"
        :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" @submit="submit" @resize="resize">
        <ef-chart class="basebg" :height="chartHeight" :padding="[24, 64, 48, 64]" :scale="[
        { dataKey: 'pass', alias: $t('fail') },
        { dataKey: 'fail', alias: $t('fail') },
        { dataKey: 'mis', alias: $t('mis') },
        { dataKey: 'rate', alias: $t('yield'), formatter: '.2%', min: 0 },
      ]" :series="[
        {
          quickType: 'bar',
          position: 'rcname*qty',
          color: 'type',
          adjust: [
            {
              type: 'dodge',
              marginRatio: 1 / 32,
            },
          ],
        },
        {
          quickType: 'line',
          position: 'rcname*rate',
        },
        {
          quickType: 'point',
          position: 'rcname*rate',
        },
      ]" :data="chartData">
        </ef-chart>
        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
            :data-source="tableData" :rowClassName="rowClassName" @change="onTableChange">
            <!--表頭title-->
            <span class="title" slot="snTitle">{{ $t('sn') }}</span>
            <span class="title" slot="rcnameTitle">{{ $t('routeCode') }}</span>
            <span class="title" slot="passTitle">{{ $t('pass') }}</span>
            <span class="title" slot="failTitle">{{ $t('fail') }}</span>
            <span class="title" slot="misTitle">{{ $t('mis') }}</span>
            <span class="title" slot="allTitle">{{ $t('all') }}</span>
            <span class="title" slot="rateTitle">{{ $t('yield') }}</span>
            <!--序號欄位顯示-->
            <template slot="sn" slot-scope="txt, rec, index">{{
                getRowNo(index)
                }}</template>
            <!--routeCode顯示-->
            <template slot="name" slot-scope="txt"><b>{{ txt }}</b></template>
            <!--pass fail mis顯示-->
            <template slot="qty" slot-scope="txt"><span v-if="txt && txt > 0">{{ txt }}</span></template>

            <!--rate顯示-->
            <template slot="rate" slot-scope="txt"><b>{{ (txt * 100).toFixed(2) + '%' }}</b></template>

            <template slot="fail" slot-scope="txt,rec">
                <a v-if="txt!=0" @click="handle_click(rec)">{{ txt }}</a>
                <!-- <a v-if="rec.key != '' && txt > 0" @click="handle_click(rec)">{{ txt }}</a>
                <span v-else>{{ txt }}</span> -->

            </template>
        </a-table>

        <defect-list :vkey="key" :paras="paras" :prop="'defect'" :title="title" :action-fix="'sfcsSMTYield'"></defect-list>

    </ef-main>
</template>

<script>
    import DataSet from '@antv/data-set'
    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import Chart from '@/components/chart/Chart'
    import { merge, isDef } from '@/utils/util'
    import DefectList from '@/pages/sfcs/smt/modal/defectList'

    const formItems = [
        {
            key: 'date',
            dataIndex: 'workTime',
            type: 'range',
            format: 'YYYY-MM-DD',
            expandDayTime: true,
            maxDays: 366,
            value: 'today',
            allowClear: false,
        },
        {
            key: 'line',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsSmt.LINE',
        },
        {
            key: 'family',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsSmt.FAMILY',
            paraMap: { title: 'name' },
        },
        {
            key: 'pn',
            type: 'popinput',
        },
        {
            key: 'wo',
            type: 'popinput',
        },
    ]
    const cols = [
        { key: 'sn', align: 'center' },
        { key: 'rcname', align: 'left' },
        { key: 'pass', slot: 'qty' },
        { key: 'fail', slot: 'fail' },
        { key: 'mis', slot: 'qty' },
        { key: 'all', slot: 'qty' },
        { key: 'rate', slot: 'rate' },
    ].map((e) => {
        e.dataIndex = e.key
        e.align = e.align || 'right'
        e.slots = { title: e.key + 'Title' }
        e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
        return e
    })

    export default {
        name: 'sfcsSMTYield',
        i18n: merge(require('@/i18n'), require('.././i18n')),
        components: { EfMain, EfChart: Chart, DefectList },
        data() {
            return {
                imageId: 'fpyimage' + new Date().format('hiu'),
                tableId: 'fpytable' + new Date().format('hiu'),
                loadcount: 0,
                sSize: 'middle',
                formItems,
                rows: [],
                sortOrder: '',
                footerRowKey: '',

                key: 0,
                paras: {},
                values: {},
                rcs: [],
            }
        },
        computed: {
            title() {
                return this.$t('defectTitle')
            },
            loading() {
                return this.loadcount > 0
            },
            chartHeight() {
                return this.sSize === 'default'
                    ? 400
                    : this.sSize === 'middle'
                        ? 300
                        : 200
            },
            chartData() {
                const { rows } = this
                const dv = new DataSet.View().source(rows)
                dv.transform({
                    type: 'fold',
                    fields: ['pass', 'fail', 'mis'],
                    key: 'type',
                    value: 'qty',
                })
                const data = dv.rows
                data.concat(
                    rows.map((e) => ({ key: e.key, rcname: e.rcname, rate: e.rate }))
                )
                return data
            },
            tableData() {
                const rows = [...this.rows]
                const ttl = {
                    key: this.footerRowKey,
                    rcname: this.$t('ttl'),
                    all: 0,
                    pass: 0,
                    fail: 0,
                    mis: 0,
                    rate: 1,
                }
                rows.forEach((e) => {
                    ttl.all += e.all
                    ttl.pass += e.pass
                    ttl.fail += e.fail
                    ttl.mis += e.mis
                    ttl.rate *= e.rate
                })
                rows.push(ttl)

                return rows
            },
            columns() {
                let rtn = cols.map((e) => {
                    e.sorter = e.key === 'sn' ? false : (a, b) => this.sorter(a, b, e.key)
                    return e
                })
                return rtn
            },
        },
        watch: {},
        created() { },
        mounted() { },
        methods: {
            submit(values) {
                this.values = values;
                this.loadData(values)
            },
            resize(size) {
                this.sSize = size
            },
            onTableChange(pagination, filters, sorter) {
                this.sortOrder = sorter.order
            },
            /**
             * 請求直通率數據
             */
            loadData(values) {
                this.loadcount += 1
                values.prop = "fpydata"
                reqPost('SFCS_SMT', values).then((res) => {
                    if (isDef(res.code) && res.code === 0) {
                        res.data.forEach((e) => (e.key = e.rcs))
                        this.rows = res.data
                        this.rcs = res.data.map(e => {
                            return e.key
                        })
                    }
                    this.loadcount -= 1
                })
            },

            rowClass(record) {
                return record.key === this.footerRowKey ? 'table-footer' : null
            },
            /**
             * 排序算法
             * 可保證匯總行(key為footerRowKey始終保持在表格最低部)
             */
            sorter(a, b, rcs) {
                const order = this.sortOrder
                const fKey = this.footerRowKey
                if (a.key === fKey) {
                    return order === 'ascend' ? 1 : -1
                } else if (b.key === fKey) {
                    return order !== 'ascend' ? 1 : -1
                } else if (rcs === 'rcname' || rcs === 'rate') {
                    const aN = a[rcs]
                    const bN = b[rcs]
                    return aN === bN ? 0 : aN > bN ? 1 : -1
                } else {
                    const aI = a[rcs] ?? 0
                    const bI = b[rcs] ?? 0
                    return Math.sign(aI - bI)
                }
            },
            getRowNo(index) {
                const len = (this.rows ?? []).length
                return index === len ? '-' : index + 1
            },
            rowClassName(record) {
                return record.key === this.footerRowKey ? 'ef-sfcs-stat-table-footer' : null
            },

            handle_click(e) {
                this.key++
                this.visible = true
                const paras = {
                    ...this.values,
                    ...{ group: '', routeCode: e.key == "" ? "FPY" : e.key, class: ['MB'] },
                }
             //   paras.status = "3";
                this.listLoading = true
                this.paras = paras
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
    .title {
        font-weight: bold;
    }

    .basebg {
        background-color: @base-bg-color;
    }
</style>