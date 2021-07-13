<template>
    <div>
        <a-modal v-model="visible" :title="title" :width="listWidth" @ok="handleOk" @cancel="handleCancel">
            <a-form :label-col="{ span: 8 }" :wrapper-col="{ span: 10 }" v-if="paras">
                <a-form-item label="Dept">
                    <a-input v-model="deptname" disabled />
                </a-form-item>
                <a-form-item label="Equip">
                    <a-input v-model="equipname" disabled />
                </a-form-item>
                <a-form-item label="From Date">
                    <a-input v-model=" fmdate" />
                </a-form-item>
                <a-form-item label="To Date">
                    <a-input v-model=" todate" />
                </a-form-item>
                <a-form-item label="Weight">
                    <a-input v-model=" weight" />
                </a-form-item>
            </a-form>
        </a-modal>
    </div>
</template>

<script>
    //  import { reqPost } from '@/services/base'

    import { merge, } from '@/utils/util'

    export default {
        name: 'iot_list',
        props: {
            actionFix: String,
            vkey: { type: Number, default: 0 },
            paras: { type: Object, default: () => ({}) },
            title: { type: String, default: "List" },
        },

        i18n: merge(require('@/i18n'), require('./i18n')),
        data() {
            return {
                loadcount: 0,
                visible: false,
                listTableWidth: 2800,
                listWidth:
                    document.documentElement.clientWidth * 0.8 > 2000
                        ? 2000
                        : document.documentElement.clientWidth * 0.8,

                deptname: "",
                equipname: "",
                fmdate: "",
                todate: "",
                weight: "",
                formvalues: {},
            }
        },
        computed: {
        },
        watch: {
            vkey() {
                console.log(this.vkey)
                this.equipname = this.paras.equipname;
                this.deptname = this.paras.name;
                this.fmdate = this.paras.fmdate;
                this.todate = this.paras.todate;
                this.weight = this.paras.weight;
                this.formvalues.dept = this.paras.dept;
                this.formvalues.equip = this.paras.equip;
                this.visible = true;
            },
        },
        created() {
            console.log(this.paras);

        },
        mounted() {

        },
        methods: {
            handleOk() {
                console.log(this.paras);
            },
            handleCancel() {
                console.log(this.paras);
            },

        },
    }
</script>