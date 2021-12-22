<template>
    <ef-main :isMainPage="true" :form-items="formItems " :openSubmit="true" @submit="submit">
    </ef-main>
</template>
<script>
    import { merge } from '@/utils/util'//,strHashColor,randomColor
    import EfMain from '@/components/page/main'


    const formItems = [
        { key: 'routeconfig', type: 'select', paraKey: 'APS.route', value: "3" },
        { key: 'family', type: 'select', mode: 'multiple', paraKey: 'APS.pjname', },
        { key: 'wo_pn', type: 'poptextarea' },
        { key: 'wo', type: 'poptextarea' },
        //工單 成品料號 route 機種多選 
    ]
    const i18n = require('./i18n')
    export default {
        name: 'workerInfo',
        components: { EfMain },
        i18n: merge(require('@/i18n'), i18n),

        data() {
            return {
                formItems,
            };
        },
        watch: {
        },
        created() {

            // this.handle_search();
        },
        methods: {
            submit(values) {
                console.log(values)
                let routeData = this.$router.resolve({ path: '/apspn',query:values});
                window.open(routeData.href, '_blank');
            }
        },
    };
</script>