<template>
  <ef-main :id="imageId" :isMainPage="true" :title="$t('fpyTitle')" :loading="loading" :openSubmit="true"
    :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" @submit="submit" @resize="resize">
    <ef-chart class="basebg" :height="chartHeight" :padding="[24, 24, 48, 64]"
      :scale="[{ dataKey: 'rate', alias: $t('fpy'), formatter: '.2%', min: 0 }]" :series="[
          {
            quickType: 'line',
            position: groupExt[groupKey] ? 'name*rate' : 'key*rate',
          },
          {
            quickType: 'point',
            position: groupExt[groupKey] ? 'name*rate' : 'key*rate',
          },
        ]" :data="fpys">
    </ef-chart>
    <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
      :data-source="tableData" :rowClassName="rowClassName" @change="onTableChange">
      <!--表頭title-->
      <span class="title" slot="snTitle">{{ $t('sn') }}</span>
      <span class="title" slot="nameTitle">{{ $t(groupKey) }}</span>
      <span class="title" slot="rateTitle">{{ $t('fpy') }}</span>
      <span class="title" v-for="rc in rcs" :key="rc.key" :slot="rc.key + 'Title'">{{ rc.name }}</span>
      <!--序號欄位顯示-->
      <template slot="sn" slot-scope="txt, rec, index">{{
        getRowNo(index)
        }}</template>
      <!--groupby顯示-->
      <template slot="name" slot-scope="txt, rec"><b>{{ txt || rec.key }}</b></template>
      <!--rate顯示-->
      <template slot="rate" slot-scope="txt">

        <b>{{ (txt * 100).toFixed(2) + '%' }}</b>
      </template>
      <!--rcs顯示-->
      <template v-for="rc in rcs" :slot="rc.key" slot-scope="txt, rec">

        <a-tooltip v-if="rec[rc.key] && rec[rc.key].all && rec[rc.key].all > 0" :key="rc.key"
          :title="getTipTitle(rec[rc.key])">
          <span v-if="rec[rc.key].rate==1 ">{{rec[rc.key] && (rec[rc.key].rate * 100).toFixed(2) + '%'}}</span>
          <a v-else @click="handle_click(rec,rc.key)">{{rec[rc.key] && (rec[rc.key].rate * 100).toFixed(2) + '%'}}</a>

          <!-- {{
            rec[rc.key] && (rec[rc.key].rate * 100).toFixed(2) + '%'
            }} -->
        </a-tooltip>
      </template>
    </a-table>

    <defect-list :vkey="key" :paras="paras" :prop="'repair'" :title="title" :action-fix="'sfcsSMTFpy'"></defect-list>
  </ef-main>
</template>

<script>
  import { reqPost } from '@/services/base'
  import EfMain from '@/components/page/main'
  import Chart from '@/components/chart/Chart'
  import { merge, isDef } from '@/utils/util'
  import DefectList from '@/pages/sfcs/smt/modal/defectList'

  const groupOption = 'line,class,family,model,pn,wo,month,week,date,hour'.split(',')
  const groupExt = {
    line: { col: 'lineName' },
    family: { col: 'familyName' },
    model: { col: 'modelName' },
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
    {
      key: 'group',
      type: 'select',
      value: 'line',
      allowClear: false,
      options: groupOption,
    },
  ]
  const preCols = [
    { key: 'sn', align: 'center' },
    { key: 'name', align: 'left' },
    { key: 'rate', align: 'right' },
  ].map((e) => {
    e.dataIndex = e.key
    e.slots = { title: e.key + 'Title' }
    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
    return e
  })

  export default {
    name: 'sfcsSMTFpy',
    i18n: merge(require('@/i18n'), require('.././i18n')),
    components: { EfMain, EfChart: Chart, DefectList },
    data() {
      return {
        imageId: 'fpyimage' + new Date().format('hiu'),
        tableId: 'fpytable' + new Date().format('hiu'),
        loadcount: 0,
        formItems,
        values: {},
        sSize: 'middle',
        fpyData: [],
        groupMap: [],
        groupExt,
        groupKey: null,
        fpys: [],
        rcs: [],
        fpy: 1,
        sortOrder: '',
        footerRowKey: '',
        formValues: null,

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
        return this.sSize === 'default'
          ? 400
          : this.sSize === 'middle'
            ? 300
            : 200
      },

      tableData() {
        const rcs = [...this.rcs]
        const data = [...this.fpys]
        const ttl = {
          key: this.footerRowKey,
          name: this.$t('ttl'),
          rate: this.fpy,
        }
        rcs.forEach((e) => {
          ttl[e.key] = e
        })
        rcs && data.push(ttl)
        return data
      },
      columns() {
        const rcs = this.rcs
        const cols = rcs.map((e) => {
          const col = {
            key: e.key,
            align: 'right',
            ellipsis: 'true',
            slots: { title: e.key + 'Title' },
            scopedSlots: { customRender: e.key },
          }
          return col
        })
        let rtn = [...preCols]
        rtn.splice(2, 0, ...cols)
        rtn = rtn.map((e) => {
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
        this.formValues = { ...values }
        const group = { key: values.group }
        this.groupKey = group.key
        const ext = this.groupExt
        if (ext[group.key] && ext[group.key].col) {
          group.name = this.groupExt[group.key].col
        }
        values.group = group
        values.order = group.name || group.key
        this.loadFpy(values)
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
      loadFpy(values) {
        this.loadcount += 1
        values.prop = "fpydata"
        reqPost('SFCS_SMT', values).then((res) => {
          console.log(res.data)
          if (isDef(res.code) && res.code === 0) {
            this.fpyData = res.data
            this.statFpy()
            this.statRcs()
          }
          this.loadcount -= 1
        })
      },

      /**
       * 统计group的數據 key,name,all,pass,mis,rate
       */
      statFpy() {
        const data = this.fpyData
        this.fpys = data.reduce(function (cr, nx) {
          const it = cr && cr.find((e) => e.key === nx.key)
          it
            ? (it.all += nx.all) &
            (it.pass += nx.pass) &
            (it.fail += nx.fail) &
            (it.mis += nx.mis) &
            (it.rate *= nx.rate) &
            (it[nx.rcs] = nx) &
            (it.name = it.name || nx.name)
            : cr.push({
              key: nx.key,
              name: nx.name,
              all: nx.all,
              pass: nx.pass,
              fail: nx.fail,
              mis: nx.fail,
              rate: nx.rate,
              [nx.rcs]: nx,
            })
          return cr
        }, [])
      },

      /**
       * 统计routeCode的數據 key:routeCode,name:routeCodeName,all,pass,mis,rate
       */
      statRcs() {
        const data = this.fpyData
        let fpy = 1
        this.rcs = data
          .reduce(function (cr, nx) {
            const it = cr && cr.find((e) => e.key === nx.rcs)
            it
              ? (it.all += nx.all) &
              (it.pass += nx.pass) &
              (it.fail += nx.fail) &
              (it.mis += nx.mis)
              : cr.push({
                key: nx.rcs,
                name: nx.rcname,
                all: nx.all,
                pass: nx.pass,
                fail: nx.fail,
                mis: nx.mis,
              })
            return cr
          }, [])
          .map((e) => {
            e.rate = e.all > e.pass + e.mis ? (e.pass + e.mis) / e.all : 1
            fpy *= e.rate
            return e
          })
        this.fpy = fpy
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
        } else if (rcs === 'name' || rcs === 'rate') {
          const aN = a[rcs]
          const bN = b[rcs]
          return aN === bN ? 0 : aN > bN ? 1 : -1
        } else {
          const aI = a[rcs].rate ?? 1
          const bI = b[rcs].rate ?? 1
          return Math.sign(aI - bI)
        }
      },
      getTipTitle(item) {
        const { pass, fail, mis, rate } = item
        const y = this.$t('yield') + ':' + (rate * 100).toFixed(2) + '%'
        const p = this.$t('pass') + ':' + pass
        const f = this.$t('fail') + ':' + fail
        const m = this.$t('mis') + ':' + mis
        return y + ' = ( ' + p + ' + ' + m + ' ) / ( ' + p + ' + ' + f + ' )'
      },
      getRowNo(index) {
        const len = (this.fpys ?? []).length
        return index === len ? '-' : index + 1
      },
      rowClassName(record) {
        return record.key === this.footerRowKey
          ? 'ef-sfcs-stat-table-footer'
          : null
      },

      handle_click(e, rc) {
        this.key++
        this.visible = true
        const { groupKey, formValues } = this
        const paras = {
          ...formValues,
          ...{ group: '', routeCode: rc, class: ['MB'] },
        }
        if (e.key != "") {
          paras[groupKey] = e.key
        }
        paras.status = "3";
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