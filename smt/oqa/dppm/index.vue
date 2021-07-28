<template>
    <ef-main :id="imageId" :isMainPage="true" :title="$t('dppmtitle')" :loading="loading" :openSubmit="true"
        :i18n="i18n" :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" @submit="submit"
        @resize="resize" :key="12">
        <v-chart :groupkey="groupKey" :groupext="groupExt" :data="tabledata"
        :action-fix="'sfcsSysDPPMSum'" :chartHeight="sSize"></v-chart>
        <v-table :groupkey="groupKey" :id="tableId" :fpys="formdata" :size="sSize" :tabledata="tabledata"
            :values="dialog_values" :rcnames="rcname" :group="group" :action-fix="'sfcsSysDPPMSum'">
        </v-table>
    </ef-main>
</template>

<script>
    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import { merge, isDef } from '@/utils/util'
    import VTable from './table_component.vue'
    import VChart from './chart_component.vue'

    const i18n = require('./i18n')
    const groupOption = [
        'line',
        'class',
        'family',
        'model',
        'pn',
        'wo',
        'month',
        'week',
        'date',
        'hour',
    ]

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
            key: 'class',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsRsbu.CLASS',
            paraMap: { title: 'name' },
            value: 'SYSTEM',
        },
        {
            key: 'wotype',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsRsbu.WOTYPE',
            paraMap: { title: 'name' },
            value: 'F',
        },
        {
            key: 'family',
            type: 'select',
            mode: 'multiple',
            paraKey: 'SfcsRsbu.FAMILY',
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
        name: 'sfcsSysDPPMSum',
        i18n: merge(require('@/i18n'), i18n),
        components: {
            EfMain,
            VTable,
            VChart,
        },
        data() {
            return {
                i18n,
                imageId: 'fpyimage' + new Date().format('hiu'),
                tableId: 'fpytable' + new Date().format('hiu'),
                loadcount: 0,
                formItems,
                values: {},
                dialog_values: {},
                sSize: 'middle',
                tabledata: [],
                groupMap: [],
                groupExt,
                groupKey: null,
                formdata: [],
                rcs: [],
                rcname: [],
                group: '',
                temp_values: {},
                fpyData: [],
                fpys: [],
                fpy: 1,
            }
        },
        computed: {
            footname() {
                return this.$t('ttl')
            },
            loading() {
                return this.loadcount > 0
            },
        },
        watch: {
            footname() {
                return this.tabledata.map((e) => {
                    return e.key ? e : (e.name = this.footname)
                })
            },
        },

        mounted() { },
        methods: {
            resize(size) {
                this.sSize = size
            },

            submit(values) {
                values.workTime[0] = values.workTime[0] + ' 00:00:00'
                values.workTime[1] = values.workTime[1] + ' 23:59:59'
                this.dialog_values = { ...values }
                const group = { key: values.group }
                this.groupKey = group.key
                const ext = this.groupExt
                if (ext[group.key] && ext[group.key].col) {
                    group.name = this.groupExt[group.key].col
                }
                values.routeCode = 'OQA'
                //values.group = group.key + (group.name ? "," + group.name : "")
                values.group = group
                values.select =
                    (group.key ? group.key + ' as KEY,' : '') +
                    (group.name ? group.name + ' as NAME,' : '')

                values.order = group.name || group.key
                this.temp_values = values
                this.loadFpy()
            },

            loadFpy() {
                var params = Object.assign({}, this.temp_values)
                params.prop = 'site'
                params.select =
                    params.select + 'oqaSample as SAMPLE,allFail as FAIL,oqaDppm as DPPM'
                reqPost('SFCS_RSBU', params).then((res) => {
                    var temp = res.data.map((e) => {
                        if (e) {
                            return {
                                name: e.name || e.key,
                                key: e.key,
                                fail: Number(e.fail),
                                dppm: Number(e.dppm),
                                sample: Number(e.sample),
                            }
                        }
                    })
                    this.loadcount -= 1
                    this.handle_mis(temp)
                })
                this.loadcount += 1
            },

            //误判
            handle_mis(element) {
                var params = Object.assign({}, this.temp_values)
                params.prop = 'repair'
                params.dutyCode = 'MISJUDGMENT'
                params.select = params.select + 'count(*) as MISS'
                reqPost('SFCS_RSBU', params)
                    .then((res) => {
                        var temp = element.map((e) => {
                            if (res.data.length > 0) {
                                // res.data.map((a) => {
                                //     if (e.key == a.key) {
                                //         e.miss = Number(a.miss)
                                //     } else {
                                //         e.miss = 0
                                //     }
                                // })
                                res.data.some(a => {
                                    if (e.key == a.key) {
                                        e.miss = Number(a.miss);
                                        return true;
                                    }
                                    else {
                                        e.miss = 0;

                                    }
                                })
                            } else {
                                e.miss = 0
                            }

                            return e
                        })

                        this.handle_yield(temp)
                        this.loadcount -= 1
                    })
                    .catch((error) => {
                        console.log(error)
                    })
                this.loadcount += 1
            },

            //直通率
            handle_yield(element) {
                var params = Object.assign({}, this.temp_values)
                this.loadcount += 1
                reqPost('SFCS_RSBU_FPYDATA', params)
                    .then((res) => {
                        if (isDef(res.code) && res.code === 0) {
                            this.fpyData = res.data
                            this.statFpy(element)
                        }

                        this.loadcount -= 1
                    })
                    .catch((error) => {
                        console.log(error)
                    })
            },

            //计算直通率
            statFpy(element) {
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

                this.tabledata = element.map((e) => {
                    this.fpys.map((a) => {
                        if (a.key == e.key) {
                            if (a.rate) {
                                e.rate = a.rate
                            } else {
                                e.rate = 0
                            }
                        }
                    })
                    e.rate = e.rate ? e.rate : 0
                    e.dppm_miss = ((e.fail - e.miss) * 1000000 / (e.sample + 0.0000001)).toFixed(0);
                    return e
                })
                this.statIO()
            },

            statIO() {
                var total_sample = 0
                var total_fail = 0
                var total_miss = 0
                this.tabledata.map((e) => {
                    total_fail = total_fail + e.fail
                    total_sample = total_sample + e.sample
                    total_miss = total_miss + e.miss
                })

                this.tabledata.push({
                    key: '',
                    name: this.footname,
                    sample: total_sample,
                    fail: total_fail,
                    miss: total_miss,
                    rate: this.statRcs(),
                    dppm:
                        ((total_fail - total_miss) / (total_sample + 0.0000001)).toFixed(6) * 1000000,
                    dppm_miss:
                        (total_fail / (total_sample + 0.0000001)).toFixed(6) * 1000000,
                })
            },

            //total_直通率
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
                return fpy
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