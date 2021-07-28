<template>
    <ef-main :id="imageId" :isMainPage="true" :title="$t('pageTitle')" :loading="loading" :openSubmit="true"
        :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" @submit="submit" @resize="resize">
        <v-chart :groupkey="groupKey" :groupext="groupExt" :action-fix="'sfcsSMTIoStat'" :data="tabledata"
            :chartHeight="sSize" :rcnames="rcname">
        </v-chart>
        <v-table :groupkey="groupKey" :id="tableId" :size="sSize" :tabledata="tabledata" :values="dialog_params"
            :rcnames="rcname" :group="group" :action-fix="'sfcsSMTIoStat'">
        </v-table>
    </ef-main>
</template>

<script>
    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'

    import { merge } from '@/utils/util'
    import VTable from './table_component.vue'
    import VChart from './chart_component.vue'

    const groupOption = ['line', 'class', 'family', 'model', 'pn', 'wo', 'month', 'week', 'date', 'hour',]
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
            format: 'YYYY-MM-DD',
            maxDays: 366,
            value: 'today',
            allowClear: false,
        },
        {
            key: 'BU',
            type: 'select',
            paraKey: 'SfcsSmt.BU',
            value: "GPI",
        },
        {
            key: 'class',
            type: 'select',
            paraKey: 'SfcsSmt.CLASS',
            value: "MB",
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

    export default {
        name: 'sfcsSMTIoStat',
        i18n: merge(require('@/i18n'), require('./i18n')),
        components: {
            EfMain,
            VTable,
            VChart,
        },
        data() {
            return {
                imageId: 'fpyimage' + new Date().format('hiu'),
                tableId: 'fpytable' + new Date().format('hiu'),
                loadcount: 0,
                formItems,
                values: {},
                sSize: 'middle',
                tabledata: [],
                groupMap: [],
                groupExt,
                groupKey: null,
                formdata: [],
                dialog_params: {},
                rcname: {},
                group: '',
                key: 0
            }
        },
        computed: {
            loading() {
                return this.loadcount > 0
            },
        },
        watch: {},
        created() {
            this.handle_column()
        },
        mounted() { },
        methods: {
            resize(size) {
                this.sSize = size
            },

            submit(values) {
                values.workTime[0] = values.workTime[0] + ' 00:00:00'
                values.workTime[1] = values.workTime[1] + ' 23:59:59'
                const group = { key: values.group }
                this.groupKey = group.key
                const ext = this.groupExt
                if (ext[group.key] && ext[group.key].col) {
                    group.name = this.groupExt[group.key].col
                }
                this.group = group.key
                //  values.group = group
                values.order = group.name || group.key
                values.group =
                    (group.key ? group.key + ',' : '') +
                    (group.name ? group.name + ',' : '') +
                    'routeCode'
                values.select =
                    (group.key ? group.key + ' as KEY,' : '') +
                    (group.name ? group.name + ' as NAME,' : '') +
                    'routeCode AS RC,sum(qty) as QTY'
                values.routeCode = 'SITEIO'
                values.type = 'O'
                this.dialog_params = values
                this.loadFpy(values)
            },

            loadFpy(values) {
                values.prop = "io";
                reqPost('SFCS_SMT', values).then((res) => {
                    this.tabledata = res.data.map((e) => {
                        e.qty = Number(e.qty != '' ? e.qty : 0)
                        e.rcname = this.rcname[e.rc]
                        return e
                    })
                    this.loadcount -= 1
                })
                this.loadcount += 1
            },
            handle_column() {
                reqPost('SFCS_SMT', { key: 'SITEIORC', prop: "consts" })
                    .then((res) => {
                        var model = res.data
                        this.rcname = model;
                    })
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