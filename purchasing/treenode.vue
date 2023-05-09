<template>
    <my-table :loading="loading" id="table11" ref="table" :tableData="treeData[0].children" :size="tableSize"
        :columns="columns" :scroll="scroll">
        <template #name="{text}">
            <b>{{text}}</b>
        </template>

        <template #search="{text,record}">
            <a v-if="record.children==undefined" @click="handle_search(record)">點擊查詢</a>
        </template>
    </my-table>


</template>

<script setup>

</script>
<script>
    import { mapState } from 'vuex'
    import { reqGet, reqPost, reqPatch, reqDelete } from '@/services/base'
    import { isDef, isEmpty, merge } from '@/utils/util'
    import MyTable from '@/components/table/index.vue'
    export default {
        name: 'basePara',
        i18n: merge(require('@/i18n'), require('./i18n')),
        components: { MyTable },
        props: {
            treeData: { type: Array, default: () => ([{ children: [] }]) }
        },
        data() {
            return {
                loading: false,
                pid: null,
                tableData: [],
                newId: -2,
                tableKey: 1,
                tableSize: 'middle',
                columns: [
                    { key: 'name', tooltip: 'true' },
                    { key: 'search', tooltip: true, title: "查詢" },

                ].map((e) => {
                    e.dataIndex = e.key
                    e.title = e.title || this.$t(e.key.toHump())
                    return e
                })
            }
        },
        computed: {
            ...mapState('setting', ['pageMinHeight', 'lang']),
            scroll() {
                return { x: this.columns.filter(item => item.visible).length * 80, y: this.pageMinHeight - 50 }
            },

        },
        methods: {
            handle_search(e) {
                // this.$emit('update:checkedKeys', e)
                console.log(e)
            },
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