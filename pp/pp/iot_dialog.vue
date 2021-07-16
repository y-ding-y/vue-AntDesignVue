<template>
    <div>
        <a-spin :spinning="loading">
            <a-modal :visible="visible" :title="title" :width="listWidth" @ok="handle_add" @cancel="handleCancel"
                :maskClosable="false">
                <a-form :label-col="{ span: 8 }" :wrapper-col="{ span: 10 }" v-if="paras">
                    <a-form-item label="Dept">
                        <a-input v-model="deptname" disabled />
                    </a-form-item>
                    <a-form-item label="Equip">
                        <a-input v-model="equipname" disabled />
                    </a-form-item>
                    <a-form-item label="From Date">
                        <a-date-picker v-model="fmdate" @change="fmdate_change" :disabled-date="disabledStartDate" />
                    </a-form-item>
                    <a-form-item label="To Date">
                        <a-date-picker v-model="todate" @change="todate_change" :disabled-date="disabledEndDate" />
                    </a-form-item>
                    <a-form-item label="Weight">
                        <a-input v-model=" weight" oninput="value=value.replace(/[^\d.]/g,'') " />
                    </a-form-item>
                </a-form>
            </a-modal>
        </a-spin>
    </div>
</template>

<script>
    import { reqPost } from '@/services/base'
    import bus from './bus.js'
    import { merge, moment } from '@/utils/util'

    export default {
        name: 'iot_list',
        props: {
            actionFix: String,
            paras: { type: Object, default: () => ({}) },
            title: { type: String, default: "List" },
            visible: { type: Boolean, default: false }
        },

        i18n: merge(require('@/i18n'), require('./i18n')),
        data() {
            return {
                loadcount: 0,
                listTableWidth: 2800,
                listWidth:
                    document.documentElement.clientWidth * 0.8 > 2000
                        ? 2000
                        : document.documentElement.clientWidth * 0.8,
                dateFormat: 'YYYY/MM/DD',
                deptname: "",
                equipname: "",
                fmdate: null,
                todate: null,
                weight: 0,
                formvalues: {},
                add_cou: 0,
            }
        },
        computed: {
            loading() {
                return this.loadcount > 0
            },
        },
        watch: {
        },
        created() {
            this.equipname = this.paras.equipname;
            this.deptname = this.paras.name;
            this.fmdate = this.paras.fmdate;
            this.todate = this.paras.todate;
            // this.weight = this.paras.weight;
            // this.formvalues.dept = this.paras.dept;
            // this.formvalues.equip = this.paras.equip;
            //  this.visible = true;
        },
        mounted() {

        },
        methods: {
            disabledStartDate(startValue) {
                return startValue.valueOf() < moment(this.paras.lastdate, 'YYYY-MM-DD');
            },
            disabledEndDate(endValue) {
                const startValue = this.fmdate;
                if (!endValue || !startValue) {
                    return false;
                }
                return moment(this.fmdate, 'YYYY-MM-DD') >= endValue;
            },
            fmdate_change(date, dateString) {
                this.fmdate = dateString || null;
            },
            todate_change(date, dateString) {
                this.todate = dateString || null;
            },
            // handleOk() {
            //     var res = 0;
            //     switch (this.paras.clicktype) {
            //         case "add":
            //             res = this.handle_add();
            //             break;
            //         case "edit":
            //             res = this.handle_edit();
            //             break;
            //     } 
            // },
            handle_add() {
                var params = {
                    "prop": "insertDist",
                    "dept": this.paras.dept,
                    "equip": this.paras.equip,
                    "fmDate": this.fmdate,
                    "toDate": this.todate,
                    "weight": String(this.weight)
                }
                for (var key in params) {
                    if (!params[key])
                        delete params[key]
                }
                if (params.fmDate > params.toDate) {
                    this.$message.warning("起始日期不能大於結束日期")
                }
                else {
                    this.loadcount = 1;
                    reqPost("EF_IOT", params).then(res => {
                        this.loadcount--;
                        if (res.data) {
                            this.add_cou++;
                            this.fmdate = null;
                            this.todate = null;
                            this.weight = 0;
                            this.$message.success("Add Success", 3)
                        }
                        else {
                            this.$message.error("Error", 3)
                        }
                    })
                }

            },
            // handle_edit() {
            //     var params = {
            //         "prop": "updateDist",
            //         "id": this.paras.recordid,
            //         "fmDate": this.fmdate,
            //         "toDate": this.todate,
            //         "weight": this.weight
            //     }
            //     console.log(params);
            //     reqPost("EF_IOT", params).then(res => {
            //         if (res.data) {
            //             this.$message.success("Update Success", 3)
            //             return 1;
            //         }
            //         else {
            //             this.$message.error("Error", 3)
            //             return -1;
            //         }
            //     })
            // },
            handleCancel() {
                bus.$emit("clicktype", {
                    _visible: false,
                    status: this.add_cou > 0 ? "ok" : "fail"
                })
            },

        },
    }
</script>