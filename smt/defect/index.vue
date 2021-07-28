<template>
  <ef-main :id="imageId" :isMainPage="true" :title="$t('defectTitle')" :loading="loading" :openSubmit="true"
    :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" :i18n="i18n" @submit="submit" @resize="resize">
    <ef-chart class="basebg" :height="chartHeight" :padding="[24, 64, 48, 64]" :scale="[
        { dataKey: 'qty', alias: $t('qty'), min: 0, max: sumQty },
        {
          dataKey: 'key',
          type: ['month', 'week', 'date', 'hour'].includes(groupKey)
            ? 'cat'
            : 'linear',
        },
        {
          dataKey: 'accQty',
          alias: $t('accQty'),
          min: 0,
          formatter: (e) => {
            return ((e * 100) / (sumQty + 0.000001)).toFixed(2) + '%'
          },
        },
      ]" :series="[
        {
          quickType: 'bar',
          position:
            groupExt[groupKey] && groupExt[groupKey].col
              ? 'name*qty'
              : 'key*qty',
        },
        {
          quickType: 'line',
          position:
            groupExt[groupKey] && groupExt[groupKey].col
              ? 'name*accQty'
              : 'key*accQty',
        },
      ]" :data="rows">
    </ef-chart>
    <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
      :data-source="tableData" :rowClassName="rowClassName" @change="onTableChange">
      <!--表頭title-->
      <span v-for="key in ['sn', 'qty', 'rate', 'accQty', 'accRate']" class="title" :slot="key + 'T'" :key="key">{{
        $t(key) }}</span>
      <span class="title" slot="nameT">{{ $t(groupKey) }}</span>
      <!--序號欄位顯示-->
      <template slot="sn" slot-scope="t, r, i">{{
        getRowSn({ t, r, i })
        }}</template>
      <!--group顯示-->
      <template slot="name" slot-scope="t, r"><b>{{ t || r.key }}</b></template>
      <template slot="qty" slot-scope="t, r"><a @click="onOpenList(r)"><b>{{ t }}</b></a></template>
      <template slot="rate" slot-scope="t, r, i">{{
        getRowRate({ t, r, i })
        }}</template>
      <template slot="accQty" slot-scope="t, r, i">{{
        getRowAccQty({ t, r, i })
        }}</template>
      <template slot="accRate" slot-scope="t, r, i">{{
        getRowAccRate({ t, r, i })
        }}</template>
    </a-table>

    <!--defect modal-->
    <defect-list :vkey="key" :paras="paras" :prop="'defect'" :title="title" :action-fix="'sfcsSMTDefect'"></defect-list>
  </ef-main>
</template>

<script>
  import { reqPost } from '@/services/base'
  import EfMain from '@/components/page/main'
  import Chart from '@/components/chart/Chart'
  import { merge, isDef, saveToExcel } from '@/utils/util'
  import DefectList from '@/pages/sfcs/smt/modal/defectList'

  const groupOption = 'line,routeCode,class,family,model,pn,wo,month,week,date,hour'.split(
    ','
  )
  const groupExt = {
    line: { col: 'lineName', order: 'lineName' },
    routeCode: { col: 'routeCodeName' },
    defectCode: { col: 'defectName' },
    family: { col: 'familyName' },
    model: { col: 'modelName' },
    month: { order: 'month' },
    week: { order: 'week' },
    date: { order: 'date' },
    hour: { order: 'hour' },
  }

  const formItems = [
    {
      key: 'date',
      dataIndex: 'workTime',
      type: 'range',
      expandDayTime: true,
      format: 'YYYY-MM-DD',
      maxDays: 366,
      value: 'today',
      allowClear: false,
    },
    {  key: 'BU', type: 'select', paraKey: 'SfcsSmt.BU', value: "GPI", allowClear: false, },
    { key: 'class', type: 'select', paraKey: 'SfcsSmt.CLASS', allowClear: false, value: "MB" },
    { key: 'line', type: 'select', mode: 'multiple', paraKey: 'SfcsSmt.LINE' },
    {
      key: 'routeCode',
      type: 'select',
      mode: 'multiple',
      paraKey: 'SfcsSmt.FPYRC',
    },
    {
      key: 'family',
      type: 'select',
      mode: 'multiple',
      paraKey: 'SfcsSmt.FAMILY',
      paraMap: { title: 'name' },
    },
    { key: 'pn', type: 'popinput' },
    { key: 'wo', type: 'popinput' },
    {
      key: 'group',
      type: 'select',
      value: 'line',
      allowClear: false,
      options: groupOption,
    },
  ]
  const cols = [
    { key: 'sn', align: 'center', width: 80 },
    { key: 'name', align: 'left' },
    { key: 'qty' },
    { key: 'rate' },
    { key: 'accQty' },
    { key: 'accRate' },
  ].map((e) => {
    e.dataIndex = e.key
    e.slots = { title: e.key + 'T', customRender: e.key }
    e.scopedSlots = e.slots
    e.align = e.align || 'right'
    return e
  })

  const sorter = (a, b, key) => (a[key] === b[key] ? 0 : a[key] > b[key] ? 1 : -1)
  const listCols = [
    { key: 'no', align: 'center', width: 50 },
    { key: 'sn', align: 'left', width: 110 },
    { key: 'bu', width: 60 },
    { key: 'class', width: 90 },
    { key: 'modelName', width: 100 },
    { key: 'pn', width: 120 },
    { key: 'wo', width: 110 },
    { key: 'lineName', width: 100 },
    { key: 'routeCodeName', width: 160 },
    { key: 'defectName' },
    { key: 'workerName', width: 80 },
    { key: 'itime', width: 150 },
    { key: 'repairName', width: 100 },
    { key: 'rootName' },
    { key: 'dutyName', width: 120 },
    { key: 'repairerName', width: 80 },
    { key: 'otime', width: 150 },
  ].map((e) => {
    e.dataIndex = e.key.toLowerCase()
    e.slots = { title: e.key + 'T', customRender: e.key }
    e.scopedSlots = e.slots
    e.align = e.align || 'left'
    e.key !== 'no' ? (e.sorter = (a, b) => sorter(a, b, e.dataIndex)) : ''
    return e
  })

  const i18n = require('./../i18n')
  export default {
    name: 'sfcsSMTDefect',
    i18n: merge(require('./../i18n'), i18n),
    components: { EfMain, EfChart: Chart, DefectList },
    data() {
      return {
        i18n,
        imageId: 'defectimage' + new Date().format('hiu'),
        tableId: 'defecttable' + new Date().format('hiu'),
        loadcount: 0,
        formItems,
        values: {},
        cols,
        sSize: 'middle',
        fpyData: [],
        groupMap: [],
        groupExt,
        groupKey: null,
        formValues: null,
        rows: [],
        sumQty: 0,
        sortOrder: '',
        footerRowKey: '',

        visible: false,
        listTableWidth: 2100,
        listWidth:
          document.documentElement.clientWidth * 0.8 > 2000
            ? 2000
            : document.documentElement.clientWidth * 0.8,
        listLoading: true,
        listId: 'defectlists' + new Date().format('hiu'),
        listData: [],
        listCols,

        key: 0,
        paras: {},
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
        return this.sSize === 'default' ? 400 : this.sSize === 'middle' ? 300 : 200
      },
      columns() {
        const { cols, groupKey, groupExt } = this
        const items = cols.filter((e) => e.key === 'qty' || e.key === 'name')
        items.forEach((e) => {
          const key =
            e.key === 'name' && !(groupExt[groupKey] && groupExt[groupKey].col)
              ? 'key'
              : e.key
          e.sorter = (a, b) => this.sorter(a, b, key)
        })
        return cols
      },
      tableData() {
        const rows = [...this.rows]
        const ttl = {
          key: this.footerRowKey,
          name: this.$t('ttl'),
          qty: this.sumQty,
        }
        rows.length > 0 && rows.push(ttl)
        return rows
      },
    },
    watch: {},
    created() { },
    mounted() { },
    methods: {
      submit(values) {
        values.prop = 'defect'
        this.formValues = { ...values }
        const group = { key: values.group }
        this.groupKey = group.key
        const ext = this.groupExt
        if (ext[group.key] && ext[group.key].col) {
          group.name = this.groupExt[group.key].col
        }
        if (ext[group.key] && ext[group.key].order) {
          values.order = this.groupExt[group.key].order
        } else {
          values.order = 'qty DESC'
        }

        //   values.class = ['MB']
        values.select = ['qty']
        values.group = group
        this.loadData(values)
      },
      resize(size) {
        this.sSize = size
      },
      onTableChange(pagination, filters, sorter) {
        this.sortOrder = sorter.order
      },
      /**
       * 查詢數據入口
       */
      loadData(values) {
        this.loadcount += 1
        reqPost('SFCS_SMT', values).then((res) => {
          if (isDef(res.code) && res.code === 0) {
            const rows = res.data
            rows.forEach((e, i) => {
              e.qty = Number(e.qty)
              e.accQty = e.qty + (i > 0 ? rows[i - 1].accQty : 0)
            })
            this.sumQty = rows[rows.length - 1]
              ? rows[rows.length - 1].accQty ?? 0
              : 0
            this.rows = rows
          }
          this.loadcount -= 1
        })
      },

      /**
       * 排序算法
       * 可保證匯總行(key為footerRowKey始終保持在表格最低部)
       */
      sorter(a, b, key) {
        const order = this.sortOrder
        const fKey = this.footerRowKey
        if (a.key === fKey) {
          return order === 'ascend' ? 1 : -1
        } else if (b.key === fKey) {
          return order !== 'ascend' ? 1 : -1
        } else {
          const aI = a[key]
          const bI = b[key]
          return aI === bI ? 0 : aI > bI ? 1 : -1
        }
      },
      getRowSn({ r, i }) {
        return r.key === this.footerRowKey ? '-' : i + 1
      },
      getRowRate({ r }) {
        return this.sumQty ? ((r.qty / this.sumQty) * 100).toFixed(2) + '%' : ''
      },
      getRowAccQty({ r, i }) {
        if (i === 0 || r.key === this.footerRowKey) {
          return r.qty
        }
        let sum = 0
        const { rows } = this
        for (let j = 0; j <= i; j++) {
          sum += rows[j].qty
        }
        return sum
      },
      getRowAccRate({ r, i }) {
        const { sumQty, getRowAccQty } = this
        return sumQty > 0
          ? ((getRowAccQty({ r, i }) / sumQty) * 100).toFixed(2) + '%'
          : ''
      },
      rowClassName(record) {
        return record.key === this.footerRowKey ? 'ant-table-stat-footer' : null
      },
      onOpenList(record) {
        // const { groupKey, formValues } = this
        // const paras = {
        //   ...formValues,
        //   ...{ group: '', class: ['SYSTEM', 'FRU'] },
        // }
        // paras[groupKey] = record.key
        // this.listData = []
        // this.listLoading = true
        // reqPost('SFCS_RSBU', paras).then((rec) => {
        //   this.listData = rec.data.map((e, i) => {
        //     e.key = i
        //     return e
        //   })
        //   this.listLoading = false
        // })
        // this.visible = true
        this.key++
        this.visible = true

        const { groupKey, formValues } = this
        const paras = {
          ...formValues,
          ...{ group: '', },
        }
        if (record.key != "") {
          paras[groupKey] = record.key
        }
        this.listLoading = true
        this.paras = paras
      },
      onSaveList() {
        saveToExcel(this.listId)
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