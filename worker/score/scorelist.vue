<template>
    <ef-main :isMainPage="true" :loading="loading" :form-items="formItems" @submit="submit">
        <a-table class="basebg"    :bordered="true" :columns="columns" :pagination="false"
            :scroll="scroll" :data-source="tabledata"    rowKey="id">
            <template slot="no" slot-scope="txt, rec, index">
                <span> {{index+1}}</span>
            </template> 

        </a-table>
    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base' 
    import { merge } from '@/utils/util'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
 

    const i18n = require('./i18n')
    export default {
        name: 'oqaworkerdstation',
        components: { EfMain },
        i18n: merge(require('./i18n'), i18n),
        data() {
            return {
                tableId: 'oqaworkerdstation' + new Date().format('hiu'),
                loadcount: 0,
                formItems,
                tabledata: [], 
                actionstatus: "",
                temp_rec: {}, 
            };
        },
        watch: {

        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { y: this.pageMinHeight - 200 }
            },
            loading() {
                return this.loadcount > 0;
            },
            columns() {
                var arr = [
                    { key: 'no', align: 'center', title: this.$t('no'), width: 100 },
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100 },
                ];

                var temp = [];
                this.stationlist.map(e => {
                    temp.push({ key: e.id, title: e.name })
                })
                console.log(temp)
                return [...arr, ...temp].map((e) => {
                    e.dataIndex = e.key
                    e.sorter = ['no', 'actions'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e
                })

            },
             
        },
        created() {
            
        },
        methods: {
            submit(values) {
                this.deptid = values.dept; 
                this.handle_search();
            }, 
            handle_search() {
                this.loadcount = 1;
                var params = {
                    prop: "getskill",
                    id: this.deptid
                }
                reqPost('WORKER_OQA', params).then(res => {
                    this.loadcount--;
                    this.tabledata =  res.data   
                })
            },

        
        },
    };
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
    }
</style>