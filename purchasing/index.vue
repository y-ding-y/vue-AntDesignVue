<template>
    <ef-main :isMainPage="true" :title="$t('pageTitle')" v-model:size="tableSize" v-model:columns="columns"
        @refresh="reload_selectedattrs">
        <template #actions>
            <a-cascader v-model:value="checkedKeys" show-search style="width: 350px;text-align:left;" :size="'small'"
                :options="treeData[0].children" placeholder="Please select"
                :field-names="{ label: 'name', value: 'id', children: 'children' }" @change="handle_change" />
            &nbsp;

            <!-- <a-tooltip title="零購料類別">
                <unordered-list-outlined @click="treenode" style="color: rgb(255, 0, 0)" />

            </a-tooltip> -->

        </template>

        <div v-for="(item,index) in attributes" style="padding:3px">
            <div style="display:flex">
                <div style="white-space:nowrap;">
                    <b>{{item.name}}：</b>
                </div>

                <div style="word-wrap:break-word; overflow:hidden">
                    <a v-for="(val,i) in get_value(item)"
                        style="margin-left:5px;letter-spacing: 1px;white-space:nowrap;"
                        @click="select_attr(item,val,index)">
                        <a-divider v-if="i!=0" style="background-color: rgb(201, 201, 201);" type="vertical" />
                        {{val}}

                    </a>

                </div>
            </div>
        </div>
        <div style="padding:3px">
            <b>篩選條件：</b>
            <template v-for="(val,index) in selectedattrs" :key="index+'_'+ikey">
                <a-tag v-if="val.val!=''" color="green" style="margin-left:15px;cursor: pointer;"
                    @click="delete_attr(index,val.val)">
                    <b> {{val.val}}</b>
                </a-tag>
            </template>
            <a-tag v-if="selectedatteval.length>0" color="orange" style="margin-left:15px;cursor: pointer;"
                @click="reload_selectedattrs">
                <reload-outlined />&nbsp;重置
            </a-tag>

        </div>

        <my-table :loading="loading" id="table11" ref="table" :tableData="filter_tableData" :size="tableSize"
            :columns="columns" :scroll="scroll">
            <template #chemical_flag="{text,record}">
                <a-tag :color="text == 'Y' ? 'green' : 'volcano'">{{ text }}</a-tag>
            </template>

        </my-table>


        <a-drawer v-model:visible="visible" placement="right" :width="drawerwidth">
            <category-tree :treeData="treeData"></category-tree>
        </a-drawer>
    </ef-main>
</template>

<script setup>

</script>
<script>
    import { mapState } from 'vuex'
    import { reqGet, reqPost, reqPatch, reqDelete } from '@/services/base'
    import { isDef, isEmpty, merge } from '@/utils/util'
    import MyTable from '@/components/table/index.vue'
    import EfMain from '@/components/page/main'
    import CategoryTree from './treenode.vue'
    export default {
        name: 'basePara',
        i18n: merge(require('@/i18n'), require('./i18n')),
        components: { EfMain, MyTable, CategoryTree },
        data() {
            return {
                loading: false,
                treeData: [{ children: [] }],
                pid: null,
                tableData: [],
                newId: -2,
                tableKey: 1,
                tableSize: 'middle',
                columns: [
                    { key: 'no', width: 50, align: 'center' },
                    { key: 'partno', width: 120, },
                    { key: 'description', width: 360, },
                    { key: 'spec', title: "SPEC", width: 260, },
                    { key: 'chemical_flag', width: 90, },
                    { key: 'unit', width: 80, },
                    { key: 'warehouse', width: 70, },
                    { key: 'create_name', width: 100, title: this.$t('create_by') },
                    { key: 'create_date', width: 160, },
                    { key: 'update_name', width: 100, title: this.$t('update_name') },
                    { key: 'update_date', width: 160, },
                ].map((e) => {
                    e.dataIndex = e.key
                    e.title = e.title || this.$t(e.key)
                    return e
                }),

                checkedKeys: '',
                attributes: [],

                ikey: 0,

                selectedatteval: [],
                dropstyle: {
                    height: '300px'
                },
                visible: false,
                drawerwidth: window.screen.width / 2
            }
        },
        watch: {
            ikey() {
                this.selectedatteval = []
                this.selectedattrs.map(e => {
                    if (e.val != '') {
                        this.selectedatteval.push(e.val)
                    }
                })

            },
        },
        computed: {
            ...mapState('setting', ['pageMinHeight', 'lang']),
            scroll() {
                return { x: this.columns.filter(item => item.visible).length * 150, y: this.pageMinHeight - 300 }
            },
            selectedattrs: {
                get: function () {
                    var res = []
                    this.attributes.map(e => {
                        res.push({
                            name: e.name,
                            val: "",
                        })
                    })
                    return res
                },
                set: function () { },
            },

            filter_tableData() {

                return this.tableData.filter(item => {
                    var flag = 1
                    this.selectedatteval.map(e => {
                        if (flag != 0)
                            flag = item.description.includes(e) ? 1 : 0
                    })
                    return flag == 1 ? true : false
                })
            },
        },
        methods: {
            treenode() {
                this.visible = true
            },
            treeDataa(source) {
                let cloneData = JSON.parse(JSON.stringify(source));
                return cloneData.filter(father => {
                    let branchArr = cloneData.filter(child => Number(father.id) === Number(child.pid));
                    branchArr.length > 0 ? (father.children = branchArr) : "";
                    return father.pid === '7543'; // 如果第一层不是parentId=0，请自行修改
                });
            },

            handle_change(value, option) {
                var description = "%";
                option.map(e => {
                    description = description + e.name + "%"
                })

                var len = value.length
                if (len > 0) {
                    this.attributes = []
                    this.$funAxios.reqPost('ERP_PURCHASING', { prop: "categoryattribute", category: value[len - 1] })
                        .then(res => {
                            var formdata = res.data.reduce(function (cr, nx) {
                                const it = cr && cr.find((e) => e.id === nx.id)
                                it
                                    ? (it[nx.value] = nx) &
                                    (it.name = it.name || nx.name)
                                    : cr.push({
                                        id: nx.id,
                                        name: nx.name,
                                        [nx.value]: nx,
                                    })
                                return cr
                            }, [])
                            this.selectedatteval = []
                            this.attributes = formdata
                        })

                    this.loading = true
                    this.tableData = []
                    this.$funAxios.reqPost('ERP_PURCHASING', { prop: "getitem", description: description })
                        .then(res => {
                            this.tableData = res.data
                            this.loading = false
                        })
                }

            },
            loadData() {
                this.loading = true
                reqGet('ERP_PURCHASING', { prop: "categorytree" }).then((res) => {
                    this.treeData = this.treeDataa(res.data)
                    this.loading = false
                })
            },

            get_value(e) {
                var res = [];
                for (var key in e) {
                    if (key != 'id' && key != 'name')
                        res.push(key)
                }
                return res
            },
            select_attr(e, val, index) {
                var temp = this.selectedattrs[index]
                temp.val = val
                this.selectedattrs.splice(index, 1, temp)
                this.ikey++
            },

            delete_attr(index, val) {
                var temp = this.selectedattrs[index]
                temp.val = ""
                this.selectedattrs.splice(index, 1, temp)
                this.ikey++
            },

            reload_selectedattrs() {
                this.selectedatteval = []
                this.selectedattrs.map(e => {
                    e.val = ""
                })
            },

            onNodeSelect(e) {
                console.log(e)
            },
        },
        mounted() {
            this.loadData()
        },
    }
</script>

<style lang="less" scoped>
    .page {
        padding: 24px;
        border-radius: 4px;
        background-color: @base-bg-color;
    }

    .title {
        text-align: center;
        letter-spacing: 1vh;
    }

    .tree {
        padding: -2px 0;
        border: 1px solid @border-color;
        border-radius: 4px;
        overflow: auto;
    }

    .table {
        background-color: @base-bg-color;
    }
</style>

<style>
    .ant-cascader-menu {
        height: 400px !important;
    }
</style>