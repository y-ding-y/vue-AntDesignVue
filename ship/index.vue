<template>
  <ef-main :id="imageId" :isMainPage="true" :title="$t('shiptitle')" :loading="loading" :openSubmit="true"
    :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" :i18n="i18n" @submit="submit" @resize="resize">
    <a-table ref="table11" class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns"
      :pagination="{ pageSize }" rowKey="id" :data-source="tableData" :rowClassName="rowClassName"
      @change="onTableChange" :scroll="{ x: 2700, y: pageMinHeight - 200 }">
      <template slot="no" slot-scope="txt, rec, index">{{ (currentPage - 1) * pageSize + index + 1 }}
      </template>

      <template slot="region" slot-scope="text, record">
        {{ handle_regionname(record.region) }}
      </template>

      <template slot="customername" slot-scope="text, record">
        <a-tooltip :title="record.customername">
          <div style=" text-overflow: ellipsis;
                    overflow: hidden ;white-space: nowrap; word-break: keep-all;">
            {{ record.customername }}
          </div>
        </a-tooltip>
      </template>

      <template slot="pn" slot-scope="text, record">
        <a-tooltip :title="record.pndesc">
          <div style=" text-overflow: ellipsis;
                    overflow: hidden ;white-space: nowrap; word-break: keep-all;">
            {{ record.pn }}
          </div>
        </a-tooltip>
      </template>

      <!-- <template v-for="item in ['familyname','inputlinename','releasedays','shipdays','inputqty','workdate']"
                :slot="item.key+'Title'">
                <a-tooltip :key="item.key">
                    <template slot="title">
                        <span v-html="item.name+': '+handle_rcname(item.rcs)"> </span>
                    </template>
                    <h4 style=" text-overflow: ellipsis;
                    overflow: hidden ;white-space: nowrap; word-break: keep-all;">{{item.name}} </h4>
                </a-tooltip>
            </template> -->
    </a-table>
  </ef-main>
</template>

<script>
  import { mapState } from 'vuex'
  import { reqPost } from '@/services/base'
  import EfMain from '@/components/page/main'
  import { merge, moment, typeOf } from '@/utils/util'

  const i18n = require('./i18n')
  const formItems = [
    {
      key: 'month',
      dataIndex: 'Time',
      type: 'month',
      format: 'YYYY-MM',
      allowClear: false,
      value: new Date().format('Y-m-d'),
    },
    {
      key: 'region',
      type: 'select',
      paraKey: 'ErpMrp.SYSSOREGION',
      value: 'gusa',
    },
    { key: 'sales', type: 'select', paraKey: 'ErpMrp.RSBUSALES' },
    { key: 'so', type: 'popinput' },
    { key: 'gg', type: 'popinput' },
  ]
  export default {
    name: 'efSysOfdShip',
    i18n: merge(require('@/i18n'), i18n),
    components: { EfMain },
    data() {
      return {
        i18n,
        imageId: 'defectimage' + new Date().format('hiu'),
        tableId: 'defecttable' + new Date().format('hiu'),
        loadcount: 0,
        sSize: 'middle',
        formItems,
        formValues: null,
        rows: [],
        sortOrder: '',
        footerRowKey: '',
        tableData: [],
        currentPage: 1,
        pageSize: 100,
        wolsit: [],
        regionnames: [],
      }
    },
    computed: {
      ...mapState('setting', ['lang', 'pageMinHeight']),
      title() {
        return this.$t('repairList')
      },
      loading() {
        return this.loadcount > 0
      },
      columns() {
        const per = [
          { key: 'no', align: 'center', fixed: 'left', width: 80 },
          { key: 'region', align: 'left', fixed: 'left', width: 80 },
          { key: 'soline', align: 'left', fixed: 'left', width: 150 },
          { key: 'salesname', align: 'left', width: 140 },
          { key: 'customername', align: 'left', width: 150 },
          { key: 'scheduleshipdate', align: 'left', width: 180 },
          { key: 'etd', align: 'left', width: 180 },
          { key: 'wo', align: 'left', width: 130 },
          { key: 'pn', align: 'left', width: 130 },
          { key: 'familyname', align: 'left', width: 100 },
          { key: 'inputlinename', align: 'left', width: 80 },
          { key: 'releasedays', align: 'right', width: 100 },
          { key: 'shipdays', align: 'right', width: 120 },
          { key: 'soqty', align: 'right', width: 90 },
          { key: 'datereleased', align: 'left', width: 180 },
          { key: 'inputqty', align: 'right', width: 80 },
          { key: 'turninqty', align: 'right', width: 100 },
          { key: 'shipqty', align: 'right', width: 90 },
          { key: 'workdate', align: 'left', width: 189 },
          { key: 'gg', align: 'left', width: 140 },
          { key: 'shiptime', align: 'left' },
        ].map((e) => {
          e.dataIndex = e.key
          e.title = this.$t(e.key)
          e.slots = { title: e.key + 'Title' }
          e.scopedSlots = { customRender: e.slot ?? e.key }
          e.sorter = e.key === 'no' ? false : (a, b) => this.sorter(a, b, e.key)
          return e
        })
        return per
      },
    },
    watch: {},
    created() {
      const params = {
        prop: 'consts',
        key: 'SYSSOREGION',
      }
      reqPost('ERP_MRP', params).then((res) => {
        this.regionnames = res.data
      })
    },
    mounted() { },
    methods: {
      submit(values) {
        this.formValues = values
        let { Time: tm, ...paras } = this.formValues
        const etd = moment(
          Number.isInteger(tm)
            ? Number(tm)
            : typeOf(tm) === 'date'
              ? tm.format('Y-m')
              : typeOf(tm) === 'object'
                ? tm.format('YYYY-MM')
                : ('' + tm).substr(0, 7) + '-01'
        )
        this.formValues.etd = paras.etd = [
          etd.startOf('month').format('Y-MM-DD HH:mm:ss'),
          etd.endOf('month').format('Y-MM-DD HH:mm:ss'),
        ]
        this.handle_solines(paras)
      },

      handle_solines(paras) {
        const prop = 'etdlines'
        this.tableData = []
        this.loadcount = 4
        reqPost('ERP_MRP', { ...paras, prop }).then((res) => {
          this.loadcount--
          this.wolsit = res.data.map((e) => {
            return e.wo
          })
          this.handle_wo(res.data)
        })
      },

      handle_wo(sos) {
        const paras = {
          prop: 'wo',
          select: ['wo', 'familyName', 'inputLineName', 'inputQty', 'turninQty'],
          wo: this.wolsit,
        }
        reqPost('SFCS_RSBU', paras).then((res) => {
          this.loadcount--
          const solines = sos.map((e) => {
            e.id = e.rowno
            res.data.some((a) => {
              if (e.wo === a.wo) {
                e = { ...e, ...a }
                return true
              }
            })
            return e
          })
          this.handle_etd(solines)
        })
      },

      handle_etd(solines) {
        const {
          etd: [fm, to],
        } = this.formValues
        const params = {
          prop: 'consts',
          key: 'ETD(' + fm + ')(' + to + ')(SYS)(GTK)',
        }

        let etdlines = []
        reqPost('ERP_MRP', params).then((res) => {
          this.loadcount--
          const etds = {}
          res.data.map((e) => {
            etds[e.solineid] = e.etd
          })
          etdlines = solines.map((e) => {
            e.etd = etds[e.solineid]
            return e
          })
          this.handle_pc(etdlines)
        })
      },

      handle_pc(etdlines) {
        const {
          etd: [fm, to],
        } = this.formValues
        const workDate =
          moment(fm)
            .subtract(4, 'days')
            .format('Y-MM-DD HH:mm:ss') +
          ',' +
          to
        const params = {
          prop: 'plan',
          select: { dt: 'MAX(date)' },
          group: ['soLineId'],
          limit: '2000',
          type: 'S',
          workDate,
          plant: 'SYS',
          product: 'SYS',
        }
        reqPost('EF_PC', params).then((res) => {
          this.loadcount--
          const soPlan = {}
          res.data.map((e) => {
            soPlan[e.solineid] = e.dt
          })
          this.tableData = etdlines.map((e) => {
            e.key = e.solineid
            e.workdate = soPlan[e.solineid] || ''
            return e
          })
        })
      },

      resize(size) {
        this.sSize = size
      },
      onTableChange(pagination, filters, sorter) {
        this.sortOrder = sorter.order
        this.currentPage = pagination.current
      },

      sorter(a, b, key) {
        const order = this.sortOrder
        const fKey = this.footerRowKey

        if (a.key === fKey) {
          return order === 'ascend' ? 1 : -1
        } else if (b.key === fKey) {
          return order !== 'ascend' ? 1 : -1
        } else if (
          [
            'releasedays',
            'shipdays',
            'soqty',
            'inputqty',
            'turninqty',
            'shipqty',
          ].includes(key)
        ) {
          const aI = Number(a[key]) || 0
          const bI = Number(b[key]) || 0
          return Math.sign(aI - bI)
        } else {
          const aI = a[key] || ''
          const bI = b[key] || ''
          return aI === bI ? 0 : aI > bI ? 1 : -1
        }
      },

      rowClassName(record) {
        return record.key === this.footerRowKey ? 'ant-table-stat-footer' : null
      },

      handle_regionname(element) {
        let res = ''
        this.regionnames.some((e) => {
          if (e.key === element) {
            res = this.lang == 'us' ? e.name_us : e.name
            return true
          }
        })
        return res === '' ? (this.lang == 'us' ? 'Other' : '其他') : res
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