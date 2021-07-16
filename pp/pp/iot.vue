<template>
    <ef-main :id="imageId" :isMainPage="true" title="" :loading="loading" :openSubmit="true" :form-items="formItems"
        :dataTableId="tableId" :imageDomId="imageId" @submit="submit" @resize="resize">

        <!-- <a-table :columns="columns1" :data-source="data" :expanded-row-keys="expandedRowKeys" @expand="onExpand" /> -->

        <a-table class="basebg" :id="tableId" :size="sSize" :bordered="true" :columns="columns"
            :pagination="{ pageSize }" :data-source="tableData" @change="onTableChange" rowKey="id" :indentSize="25"
            :scroll="scroll" :expanded-row-keys="expandedRowKeys" @expand="onExpand">

            <span v-for="item in cols" :slot="item.key+'Title'" :key="item+'Title'">
                <a-tooltip :key="item+'Title'">
                    <template slot="title">
                        {{ $t('name')}}： {{item.name}}<br> {{$t('addr')}}：{{item.addr}}<br>
                        {{ $t('ip')}}：{{item.ip}}<br> key：{{item.key}}
                    </template>
                    <h4 style=" text-overflow: ellipsis;
                              overflow: hidden ;white-space: nowrap; word-break: keep-all;">
                        {{ item.name.replace(item.name.split(" ")[0], "")|| item.name }}
                    </h4>
                </a-tooltip>
            </span>

            <template v-for="item in cols" :slot="item.key" slot-scope="text, record">
                <!-- {{ record[item.key].split(',')[1] }}~{{ record[item.key].split(',')[2] }}
               
                {{ record[item.key].split(',')[0] }} -->
                <!-- {{record[item.key]}}  -->
                <div :key="item.key+'span'" v-if="record[item.key]">
                    {{ record[item.key][0].fmdate}}~{{ record[item.key][0].todate || '--'}}
                    <br>
                    <b> {{ record[item.key][0].weight}}</b>
                    <!-- <a-button :key="item.key+'btnedit'" :class="'warning'" type="link" size="small"
                            @click="handle_dialog('edit',item.key,item.name,record)">
                            <a-icon type="edit" />
                        </a-button> -->

                    <a-button :key="item.key+'btn'" :class="!record[item.key][0].todate?'success2':'success'"
                        type="link" size="small" :disabled="!record[item.key][0].todate"
                        @click="handle_dialog('add',item.key,item.name,record)">
                        <a-icon type="plus" />
                    </a-button>


                    <a-button :key="item.key+'btndetail'" :class="'warning'" type="link" size="small" shape="circle"
                        @click="handle_detail( item.key,item.name,record)">
                        <a-icon type="ellipsis" />
                    </a-button>

                    <span :class="'warning'">({{record[item.key].length}})</span>

                </div>
                <a-button v-else :key="item.key+'btn'" :class="'success'" type="link" size="small"
                    @click="handle_dialog('add',item.key,item.name,record)">
                    <a-icon type="plus" />
                </a-button>

            </template>

            <template slot="no" slot-scope="txt, rec ,index">
                <a-tooltip>
                    <template slot="title">
                        id：{{rec.id}}
                    </template>
                    <b> {{ (currentPage - 1) * pageSize + index + 1 }}.&nbsp;{{rec.name}}({{rec.count}})</b>
                </a-tooltip>
            </template>
        </a-table>
        <v-iot v-if="visible" :key='i' :visible="visible" :paras="paras" :title="dialogtitle[clicktype]"
            :action-fix="'iot'">
        </v-iot>

        <v-detail v-if="visible_detail" :key='i_detail' :visible="visible_detail" :paras="detailtable" :title="'Detail'"
            :action-fix="'iot'">
        </v-detail>
    </ef-main>
</template>

<script>

    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    import { merge, isDef } from '@/utils/util'
    import VIot from './iot_dialog'
    import bus from './bus.js'
    import VDetail from './iot_detail'

    const formItems = [
        {
            key: 'type',
            type: 'select',
            paraKey: 'SfcsPp.TYPE',
            value: "101"
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
    ]


    export default {
        name: 'iot',
        i18n: merge(require('@/i18n'), require('./i18n')),
        components: { EfMain, VIot, VDetail },
        data() {
            return {
                imageId: 'fpyimage' + new Date().format('hiu'),
                tableId: 'fpytable' + new Date().format('hiu'),
                loadcount: 0,
                sSize: 'middle',
                formItems,
                vkey: 0,
                i: 0,
                i_detail: 0,
                sortOrder: '',
                footerRowKey: '',
                tableData: [],
                currentPage: 1,
                pageSize: 100,
                paras: {},
                visible: false,
                visible_detail: false,
                formValues: {},
                cols: [],
                cols_first: [],
                distdata: [],

                depts: [],
                equips: [],
                detailtable: [],

                expandedRowKeys: [],


                clicktype: "",
                dialogtitle: {
                    add: "Add",
                    edit: "Edit",
                }
            }
        },
        computed: {
            scroll() {
                return { x: this.cols.length * 180 + 100, y: this.pageMinHeight - 200 }
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),
            title() {
                return this.$t('detailTitle')
            },
            loading() {
                return this.loadcount > 0
            },

            columns() {
                var arr = [
                    {
                        key: 'no', align: 'left', width: 200, title: this.$t('no'), fixed: "left",
                        scopedSlots: { customRender: 'no' },
                    },
                ]
                var temp = [];
                var arr_cols = []
                if (this.formValues.type == "101") {
                    arr_cols = this.dedupe(this.cols_first).map(e => {
                        return {
                            name: e
                        }
                    })
                    arr_cols.map((e, i) => {
                        e.key = "e" + i;
                        e.title = e.name;
                        e.children = [];
                        e.align = "left";
                        this.cols.map(a => {
                            if (e.name == a.name.split(" ")[0]) {
                                e.children.push({
                                    key: a.key,
                                    //   title: a.name.replace(a.name.split(" ")[0], ""),
                                    dataIndex: a.key,
                                    slots: { title: a.key + 'Title' },
                                    scopedSlots: { customRender: a.slot ?? a.key }
                                })
                            }
                        })
                        return e;
                    })
                    temp = [...arr, ...arr_cols]
                }
                else {
                    this.cols.map(e => {
                        e.dataIndex = e.key
                        //   e.title = e.name
                        e.slots = { title: e.key + 'Title' }
                        e.scopedSlots = { customRender: e.slot ?? e.key }
                        return e
                    });
                    temp = [...arr, ...this.cols]
                }
                return temp;
            },
        },
        watch: {},
        created() {
            bus.$on("clicktype", msg => {
                this.visible = msg._visible;
                this.visible_detail = msg.detail_visible
                if (msg.status == "ok") {
                    this.submit(this.formValues);
                }
            })
        },
        mounted() { },
        methods: {
            onExpand(expanded, record) {
                if (expanded) {
                    // 设置展开窗Key，代表展开操作
                    this.expandedRowKeys.push(record.id)
                } else {
                    // 代表折叠操作
                    if (this.expandedRowKeys.length) {
                        this.expandedRowKeys = this.expandedRowKeys.filter(v => {
                            return v !== record.id
                        })
                    }
                }
            },

            submit(values) {
                this.tableData = [];
                this.depts = [];
                this.equips = [];
                this.loadcount = 3;
                this.formValues = { ...values }
                this.handle_search(values)
            },
            handle_consts() {
                var params = {
                    prop: "consts",
                    key: "GTKDEPTTREES",
                };
                reqPost("EF_IOT", params)
                    .then(res => {
                        this.loadcount--;
                        //  this.tableData = res.data;
                        // this.depts = res.data.map(e => {
                        //     return e.id
                        // })
                        this.routeExpandAll(res.data)
                        isDef(this.depts) && this.handle_data(res.data);
                    })
            },

            routeExpandAll(resdata) {
                var res = [];
                resdata.map((e) => {
                    isDef(e.children) && this.routeExpandAll(e.children)
                    res.push(e.id)

                })
                this.depts = [...this.depts, ...res]
            },

            handle_search(values) {
                reqPost("SFCS_PP", { ...values, prop: "equip", orgId: "476", order: "name" })
                    .then(res => {
                        this.loadcount--;
                        this.cols = res.data;
                        this.cols_first = res.data.map(e => {
                            this.equips.push(e.key)
                            return e.name.split(" ")[0]
                        })
                        this.handle_consts();
                    })
            },

            //数组去重
            dedupe(array) {
                return Array.from(new Set(array));
            },


            handle_data(element) {
                var params = {
                    prop: "dist",
                    dept: this.depts,
                    equip: this.equips,
                    order: "dateFrom desc"
                };
                reqPost("EF_IOT", params)
                    .then(res => {
                        this.loadcount--;
                        this.distdata = res.data;
                        var ss = []
                        this.tableData = element.map(e => {
                            this.recursion(e, ss)
                            return e;
                        })
                        ss.map(e => {
                            var count = 0;
                            res.data.map(a => {
                                if (e.id == a.dept) {
                                    count++;
                                    // e[a.equip] = a.fmdate + "~" + a.todate + "</br><b>" + a.weight + "<b>";
                                    // e[a.equip].push({
                                    //     weight: a.weight,
                                    //     fmdate: a.fmdate,
                                    //     todate: a.todate,
                                    // });
                                    e[a.equip] = [...e[a.equip] ?? [], ...[{
                                        weight: a.weight,
                                        fmdate: a.fmdate,
                                        todate: a.todate,
                                        recordid: a.id,
                                    }]]
                                }

                            })
                            e.count = count;
                            return e;
                        })
                    })
            },


            recursion(node, arr) {
                if (!node.children) {
                    arr.push(node)
                } else {
                    arr.push(node)
                    node.children.forEach(item => {
                        this.recursion(item, arr)
                    })
                }
            },

            handle_dialog(e, equip, equipname, rec) {
                this.vkey++
                this.i++;
                this.clicktype = e;
                this.visible = true;
                var params = {
                    clicktype: e,
                    equipname: equipname,
                    equip: equip,
                    dept: rec.id,
                    deptname: rec.name,
                    lastdate: rec[equip][0].fmdate || null,
                    // fmdate: e == "edit" ? rec[equip][0].fmdate : "",
                    // todate: e == "edit" ? rec[equip][0].todate : "",
                    // weight: e == "edit" ? rec[equip][0].weight : "",
                    fmdate: null,
                    todate: null,
                    weight: "",
                }
                this.paras = { ...params, ...rec };
            },

            handle_detail(equip, equipname, rec) {
                this.i_detail++;
                this.visible_detail = true;
                this.detailtable = rec[equip].map(e => {
                    return {
                        id: e.recordid,
                        equip: equip,
                        equipname: equipname,
                        dept: rec.id,
                        deptname: rec.name,
                        fmDate: e.fmdate,
                        toDate: e.todate,
                        weight: e.weight
                    }
                })
            },
            resize(size) {
                this.sSize = size
            },
            onTableChange(pagination, filters, sorter) {
                this.sortOrder = sorter.order
                this.currentPage = pagination.current
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

    .success,
    .success:hover,
    .success:visited {
        /* background-color: @success-color; */
        color: @success-color;
        float: right;
    }

    .success2,
        {
        /* background-color: @success-color; */
        color: grey;
        float: right;
    }

    .warning,
    .warning:hover,
    .warning:visited {
        /* background-color: @success-color; */
        color: @error-color;
        float: right;
    }
</style>