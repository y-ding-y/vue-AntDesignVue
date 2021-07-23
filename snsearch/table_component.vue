<template>
    <div>
        <a-table class="basebg" :id="listId" :size="'small'" :bordered="true" :columns="listCols" :pagination="false"
            :data-source="listData" :loading="listLoading">
            <span slot="noT" class="title">{{ $t('sn') }}</span>
            <template slot="no" slot-scope="t, r, i">{{ i + 1 }}</template>
        </a-table>
    </div>
</template>

<script>
    import { reqPost } from '@/services/base'
    import { merge, isDef } from '@/utils/util'

    const i18n = require('./i18n')
    export default {
        name: 'snview_table',
        props: {
            actionFix: String,
            sn: String,
            types: String,
            cols: Array,
            order: String,
            api: String,
        },
        i18n: merge(require('@/i18n'), i18n),

        data() {
            return {
                i18n,
                loadcount: 0,
                listTableWidth: 1100,
                listLoading: false,
                listId: 'defectlists' + new Date().format('hiu'),
                listData: [],
            }
        },
        computed: {
            loading() {
                return this.loadcount > 0
            },

            listCols() {
                return [...[
                    { key: 'no', align: 'center', width: 50 },
                ].map((e) => {
                    e.dataIndex = e.key.toLowerCase()
                    e.slots = { title: e.key + 'T', customRender: e.key }
                    e.scopedSlots = e.slots
                    return e
                }), ...this.cols]
            }
        },
        created() {
            this.onOpenList();
        },
        methods: {
            onOpenList() {
                var params = {
                    prop: this.types,
                    sn: this.sn,
                    order: this.order
                };
                this.listLoading = true;
                reqPost(this.api, params).then((rec) => {
                    this.listLoading = false;
                    if (isDef(rec.data)) {
                        this.listData = rec.data.map((e, i) => {
                            e.key = i
                            return e;
                        })
                    }

                })
            },

        },
    }
</script>