<template>
  <ef-main :id="imageId" style="min-height:380px;" :isMainPage="false" :title="$t('searchetd')" :loading="loading"
    :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" @submit="submit" @resize="resize">
    <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns" rowKey="rowno"
      :pagination="false" :scroll="{ y: tableheight }" :data-source="tableData" @change="onTableChange">
      <template slot="no" slot-scope="txt, rec, i">
        {{ i + 1 }}
      </template>

      <template slot="confirmation" slot-scope="text, record">
        <a-tooltip :title="record.confirmation">
          <div style="text-overflow: ellipsis;overflow: hidden ;white-space: nowrap; word-break: keep-all;">
            {{ record.confirmation }}
          </div>
        </a-tooltip>
      </template>
    </a-table>
  </ef-main>
</template>

<script>
  import { reqPost } from '@/services/base'
  import EfMain from '@/components/page/main'
  import { merge } from '@/utils/util'
  import { mapState } from 'vuex'

  const formItems = [
    {
      key: 'date',
      dataIndex: 'iTime',
      type: 'range',
      expandDayTime: true,
      format: 'YYYY-MM-DD',
      maxDays: 31,
      value: 'today',
      allowClear: false,
    },
  ]
  export default {
    name: 'efetdsearch',
    i18n: merge(require('@/i18n'), require('./i18n')),
    components: { EfMain },
    data() {
      return {
        imageId: 'fpyimage' + new Date().format('hiu'),
        tableId: 'fpytable' + new Date().format('hiu'),
        loadcount: 0,
        sSize: 'middle',
        sortOrder: '',
        footerRowKey: '',
        formItems,
        listid: [],
        tableData: [],
        tabPosition: 'left',
        woactived: '',
        wolist: [],
        listData: [],
        updatetime: '',
      }
    },
    computed: {
      tableheight() {
        let h = this.pageMinHeight - 250
        return h < 250 ? 250 : h
      },
      ...mapState('setting', ['pageMinHeight']),

      loading() {
        return this.loadcount > 0
      },
      columns() {
        var arr = [
          { key: 'no', align: 'center', width: 50 },
          { key: 'pn', align: 'left' },
          { key: 'description', align: 'left' },
          { key: 'cn_description', align: 'left' },
          { key: 'wkdate', align: 'left', width: 150 },
          { key: 'confirmation', align: 'left', width: 150 },
          { key: 'createdby', align: 'left' },
          { key: 'createdat', align: 'left', width: 150 },
        ].map((e) => {
          e.dataIndex = e.key.toLowerCase()
          e.title = this.$t(e.key)
          e.slots = { title: e.key, customRender: e.key }
          e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
          e.sorter = e.key === 'no' ? false : (a, b) => this.sorter(a, b, e.key)
          return e
        })
        return arr
      },
    },
    watch: {},
    created() { },
    mounted() { },
    methods: {
      submit(values) {
        this.handle_search(values)
      },

      handle_search(values) {
        var params = {
          prop: 'etd',
          limit: '1000',
          flag: 'Y',
          etd: values.iTime,
        }
        this.loadcount = 1
        reqPost('EF_PC', params).then((res) => {
          this.loadcount--
          this.tableData = res.data
        })
      },

      resize(size) {
        this.sSize = size
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
        } else {
          const aN = a[rcs] ?? ''
          const bN = b[rcs] ?? ''
          return aN === bN ? 0 : aN > bN ? 1 : -1
        }
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