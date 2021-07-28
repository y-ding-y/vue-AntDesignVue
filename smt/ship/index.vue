<template>
  <ef-main :id="imageId" :isMainPage="true" :title="$t('shipTitle')" :loading="loading" :openSubmit="true"
    :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" @submit="submit" @resize="resize">
    <ef-chart class="basebg" :height="chartHeight" :padding="[24, 24, 48, 64]" :scale="[{ dataKey: 'qty' }]" :series="[
        {
          quickType: 'bar',
          position: 'key*qty',
          color: 'class',
          adjust: [
            {
              type: 'dodge',
              marginRatio: 1 / 32,
            },
          ],
        },
      ]" :data="tempdata">
    </ef-chart>

    <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
      :data-source="tabledata" :rowClassName="rowClassName" @change="onTableChange">
      <template slot="no" slot-scope="txt, rec, index">{{
        getRowNo(index)
        }}</template>

      <!-- <template slot="qty" slot-scope="txt, rec">
				<a @click="handle_dialog(rec,'')">{{txt}}</a>
			</template> -->
      <template slot="name" slot-scope="txt, rec"><b>{{ txt || rec.key }}</b></template>

      <template slot="system" slot-scope="txt, rec">
        <!-- {{ txt || rec.key }} -->
        <a v-if="rec.key != '' && handle_search(rec.key, 'SYSTEM') > 0" @click="handle_dialog(rec, 'SYSTEM')">{{
          handle_search(rec.key, 'SYSTEM') }}</a>
        <a v-if="rec.key == '' && txt > 0" @click="handle_dialog(rec, 'SYSTEM')">{{ txt }}</a>
      </template>

      <template slot="fru" slot-scope="txt, rec">
        <!-- {{ txt || rec.key }} -->
        <a v-if="rec.key != '' && handle_search(rec.key, 'FRU') > 0" @click="handle_dialog(rec, 'FRU')">{{
          handle_search(rec.key, 'FRU') }}</a>
        <a v-if="rec.key == '' && txt > 0" @click="handle_dialog(rec, 'FRU')">{{
          txt
          }}</a>
      </template>
    </a-table>
    <v-dialog :vkey="key" :paras="paras" :prop="'ship'" :title="title" :action-fix="'sfcssmtShip'"></v-dialog>
  </ef-main>
</template>

<script>
  import { reqPost } from '@/services/base'
  import EfMain from '@/components/page/main'
  import { merge } from '@/utils/util'
  import VDialog from './dialog.vue'
  import Chart from '@/components/chart/Chart'

  const groupOption = 'family,model,pn,month,week,date,hour'.split(',')
  const groupExt = {
    family: { col: 'familyName' },
    model: { col: 'model' },
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
      key: 'gg',
      type: 'popinput',
    },
    {
      key: 'group',
      type: 'select',
      value: 'model',
      allowClear: false,
      options: groupOption,
    },
  ]

  export default {
    name: 'sfcsSMTShip',
    i18n: merge(require('@/i18n'), require('./i18n')),
    components: { EfMain, VDialog, EfChart: Chart },
    data() {
      return {
        imageId: 'fpyimage' + new Date().format('hiu'),
        tableId: 'fpytable' + new Date().format('hiu'),
        loadcount: 0,
        formItems,
        groupKey: null,
        groupExt,
        values: {},
        sSize: 'middle',
        tabledata: [],
        sortOrder: '',
        footerRowKey: '',
        formValues: null,
        key: 0,
        paras: {},
        tempdata: [],
        temp_total: {},
      }
    },
    computed: {
      chartHeight() {
        return this.sSize === 'default'
          ? 400
          : this.sSize === 'middle'
            ? 300
            : 200
      },
      footname() {
        return this.$t('ttl')
      },
      title() {
        return this.$t('shipdialog')
      },
      loading() {
        return this.loadcount > 0
      },
      columns() {
        return [
          { key: 'no', align: 'center', title: this.$t('no') },
          {
            key: 'name',
            align: 'left',
            width: 200,
            title: this.$t(this.groupKey),
          },
          { key: 'system', align: 'right', title: 'SYSTEM' },
          { key: 'fru', align: 'right', title: 'FRU' },
          //		{ key: 'qty', align: 'right', title: this.$t("qty") },
        ].map((e) => {
          e.dataIndex = e.key
          e.sorter = e.key === 'no' ? false : (a, b) => this.sorter(a, b, e.key)
          e.slots = { title: this.$t(e.key) }
          e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
          return e
        })
      },
    },
    watch: {
      footname() {
        return this.tabledata.map((e) => {
          if (e.key == '') {
            e.name = this.footname
          }
          return e
        })
      },
    },
    methods: {
      submit(values) {
        const group = { key: values.group }
        this.groupKey = group.key
        const ext = this.groupExt
        if (ext[group.key] && ext[group.key].col) {
          group.name = this.groupExt[group.key].col
        }
        values.group =
          (group.key ? group.key : '') + (group.name ? ',' + group.name : '')
        values.select =
          (group.key ? group.key + ' as KEY,' : '') +
          (group.name ? group.name + ' as NAME,' : '') +
          ' sum(qty) as QTY '
        values.shipDate = values.workTime
        this.formValues = { ...values }
        this.handle_ship(values)
      },

      handle_ship(values) {
        values.prop = 'ship'
        this.loadcount += 1
        reqPost('SFCS_SMT', values).then((res) => {
          var total_qty = 0
          res.data.map((e) => {
            total_qty = total_qty + Number(e.qty)
          })
          this.temp_total = { key: '', name: this.footname, qty: total_qty }
          this.handle_type(values)
          this.tabledata = [
            ...res.data.map((e) => {
              e.qty = Number(e.qty)
              return e
            }),
          ]
          this.loadcount -= 1
        })
      },

      handle_type(values) {
        values.group = values.group + ',class'
        values.select = values.select + ',class AS CLASS'
        values.prop = 'ship'
        this.loadcount += 1
        reqPost('SFCS_SMT', values).then((res) => {
          var total_sys = 0
          var total_fru = 0
          res.data.map((e) => {
            e.qty = Number(e.qty)
            if (e.class === 'SYSTEM') {
              total_sys = total_sys + Number(e.qty)
            }
            if (e.class === 'FRU') {
              total_fru = total_fru + Number(e.qty)
            }
            return e
          })
          this.temp_total.system = total_sys
          this.temp_total.fru = total_fru
          this.tabledata = [...this.tabledata, this.temp_total]
          this.loadcount -= 1
          this.tempdata = res.data
        })
      },

      handle_search(element, type) {
        var res = 0
        this.tempdata.map((e) => {
          if (e.key == element && e.class == type) {
            res = Number(e.qty)
          }
        })
        return res
      },

      resize(size) {
        this.sSize = size
      },
      onTableChange(pagination, filters, sorter) {
        this.sortOrder = sorter.order
      },

      rowClass(record) {
        return record.key === this.footerRowKey ? 'table-footer' : null
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

      getRowNo(index) {
        const len = (this.tabledata ?? []).length
        return index === len - 1 ? '-' : index + 1
      },
      rowClassName(record) {
        return record.key === this.footerRowKey
          ? 'ef-sfcs-stat-table-footer'
          : null
      },
      handle_dialog(e, type) {
        this.key++
        this.visible = true
        const { groupKey, formValues } = this
        const paras = {
          ...formValues,
        }
        delete paras.select
        delete paras.group
        delete paras.workTime
        paras[groupKey] = e.key
        paras.class = type
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