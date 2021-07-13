<template>
    <ef-main :id="imageId" :isMainPage="true" :title="$t('pageTitle')" :loading="loading" :openSubmit="true"
        :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" @submit="submit" @resize="resize">

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
      
                <div :key="item.key+'span'" v-if="record[item.key]">
                    {{record[item.key][0].fmdate}}~{{record[item.key][0].todate ||''}}
                    <br>

                    <b>{{record[item.key][0].weight}}</b>
                    <a-button :key="item.key+'btn'" :class="'warning'" type="link" size="small"
                        @click="handle_dialog('edit',item.key,item.name,record)">
                        <a-icon type="edit" />
                    </a-button>
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
        <v-iot :vkey="vkey" :paras="paras" :title="dialogtitle[clicktype]" :action-fix="'iot'">
        </v-iot>

    </ef-main>
</template>

<script>

    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    import { merge } from '@/utils/util'
    import VIot from './iot_dialog'

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
        components: { EfMain, VIot },
        data() {
            return {
                imageId: 'fpyimage' + new Date().format('hiu'),
                tableId: 'fpytable' + new Date().format('hiu'),
                loadcount: 0,
                sSize: 'middle',
                formItems,
                vkey: 0,
                sortOrder: '',
                footerRowKey: '',
                tableData: [],
                currentPage: 1,
                pageSize: 100,
                paras: {},
                visible: false,
                formValues: {},
                cols: [],
                cols_first: [],

                depts: [],
                equips: [],

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
                if (this.formValues.type === "101") {
                    return { x: 8600, y: this.pageMinHeight - 200 }
                }
                else {
                    return { x: 4000, y: this.pageMinHeight - 200 }
                }
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

        },
        mounted() { },
        methods: {
            onExpand(expanded, record) {
                if (expanded) {
                    // 设置展开窗Key，代表展开操作
                    this.expandedRowKeys.push(record.id)
                    console.log(record);
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
                        this.depts = res.data.map(e => {
                            return e.id
                        })
                        this.handle_data(res.data);
                    })
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
                    equip: this.equips
                };
                reqPost("EF_IOT", params)
                    .then(res => {
                        this.loadcount--;
                        this.tableData = element.map(e => {
                            var count=0;
                            res.data.map(a => {
                                if (e.id == a.dept) {
                                    count++;
                                    // e[a.equip] = a.fmdate + "~" + a.todate + "</br><b>" + a.weight + "<b>";
                                    e[a.equip] = [{
                                        weight: a.weight,
                                        fmdate: a.fmdate,
                                        todate: a.todate,
                                    }];
                                }
                            })
                            e.count=count;
                            return e;
                        })
                    })
            },

            handle_dialog(e, equip, equipname, rec) {
                this.vkey++
                this.clicktype = e;
                var params = {
                    clicktype: e,
                    equipname: equipname,
                    equip: equip,
                    dept: rec.id,
                    deptname: rec.name,
                    fmdate: rec[equip][0].fmdate,
                    todate: rec[equip][0].todate,
                    weight: rec[equip][0].weight,
                }

                console.log(params);
                this.paras = { ...params, ...rec };
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
    }

    .warning,
    .warning:hover,
    .warning:visited {
        /* background-color: @success-color; */
        color: @warning-color;
        float: right;
    }
</style>
