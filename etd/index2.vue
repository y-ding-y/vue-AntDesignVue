<template>
    <ef-main :isMainPage="true" :title="$t('etdtitle')" :i18n="i18n" @resize="resize" :loading="loading">
      <a-textarea v-model="value" :placeholder="$t('content')" :auto-size="{ minRows: 3, maxRows: 5 }" />
      <a-table class="basebg" :size="sSize" :bordered="true" :columns="columns" :pagination="false"
        :data-source="tableData" :scroll="{ y: 200 }" @change="onTableChange" rowKey="id"><template slot="no"
          slot-scope="txt, rec, index">{{ index + 1 }}
        </template>
      </a-table>
      <div style="width: 100%; display: flex;justify-content:flex-end;">
        <!-- <a-button @click="handle_save" type="default">OK</a-button> -->
        &nbsp;&nbsp;&nbsp;
        <a-button @click="handle_upload" type="primary">{{
          $t('upload')
          }}</a-button>
      </div>
    </ef-main>
  </template>
  
  <script>
    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
  
    import { merge } from '@/utils/util'
  
    const i18n = require('./i18n')
    export default {
      name: 'efSysOfdEtd',
      i18n: merge(require('@/i18n'), i18n),
      components: { EfMain },
      data() {
        return {
          i18n,
          sSize: 'small',
          rows: [],
          loadcount: 0,
          value: '',
          tableData: [],
        }
      },
      computed: {
        loading() {
          return this.loadcount > 0
        },
        columns() {
          var per = [
            { key: 'No.', align: 'center', slot: 'no', width: 50 },
            { key: 'mtcSo', align: 'center', width: 120 },
            { key: 'wkDate', align: 'left', width: 100 },
            { key: 'confirmation', align: 'left', ellipsis: true },
            { key: 'soLineId', align: 'left', width: 120 },
            { key: 'deliveryDetailId', align: 'left', width: 120 },
          ].map((e) => {
            e.dataIndex = e.key
            e.title = e.key
            e.slots = { title: e.key + 'Title' }
            e.scopedSlots = { customRender: e.slot ?? e.key }
            //  (e.sorter = e.key === 'no' ? false : (a, b) => this.sorter(a, b, e.key));
            return e
          })
          return per
        },
      },
      watch: {
        value() {
          this.handle_save()
        },
      },
      created() { },
      mounted() { },
      methods: {
        handle_save() {
          this.loadcount = 1
          var res = []
          // var source = $("#cChangReasons").val();//获取文本内容
          var rows = this.value.split('\n') //换行分割
          for (var i = 0; i < rows.length; i++) {
            var row = rows[i].split('\t') //空格分割
            if (row.length > 5) {
              res.push({
                mtcSo: row[0].trim(),
                wkDate: row[3].trim(),
                confirmation: row[6].trim(),
                soLineId: row[9].trim(),
                deliveryDetailId: row[12].trim(),
              })
            }
          }
          this.loadcount--
          this.tableData = res
        },
  
        handle_upload() {
          var params = {
            prop: 'insertEtd',
            site: 'GTK',
            rows: this.tableData,
          }
          this.loadcount = 1
          reqPost('EF_PC_INSERTETD', params).then((res) => {
            this.loadcount--
            if (res.data) {
              this.$message.success('Success')
              this.handle_clear()
            } else {
              this.$message.error('Fail')
            }
          })
        },
        handle_clear() {
          this.tableData = []
          this.value = ''
        },
        resize(size) {
          this.sSize = size
        },
        onTableChange(pagination, filters, sorter) {
          this.sortOrder = sorter.order
        },
  
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
      },
    }
  </script>
  
  <style scoped lang="less">
    .basebg {
      background-color: @base-bg-color;
    }
  </style>