<template>
    <ef-main :isMainPage="true" :loading="loading" :goback="true">
        <div style="display: flex;width: 30%;justify-content: space-between;">
            <div class="top2_title " style="width: 45%; border: 1px solid; ">
                工號：{{empno}}

            </div>
            <div class="top2_title " style="width: 45%; border: 1px solid ">
                姓名：{{name}}
            </div>
        </div>

        <br>
        <div class="top">
            <div class="top1">
                <div class="top1_title" @click="handle_skill"> 技能 </div>
                <div class="top1_content">
                    <div class="top2_title"> Total崗位數 </div>
                    <div class="top2_content  colorlinear_green"> {{totalstation}} </div>
                </div>
                <div class="top1_content">
                    <div class="top2_title"> 考核Pass </div>
                    <div class="top2_content  colorlinear_green"> {{passstation}} </div>
                </div>
                <div class="top1_content">
                    <div class="top2_title" style="height: 80px;line-height: 80px"> 未考核數 </div>
                    <div class="top2_content  colorlinear_green" style="height: 80px;line-height: 80px; ">
                        {{failstation}}
                    </div>
                </div>

            </div>
            <div class="top1">
                <div class="top1_title" @click="handle_erp"> 出勤 </div>
                <div class="top1_content">
                    <div class="top2_title"> 工作天數 </div>
                    <div class="top2_content colorlinear_blue"> {{ondutydays}} </div>
                </div>
                <div class="top1_content">
                    <div class="top2_title"> 加班小時</div>
                    <div class="top2_content colorlinear_blue"> {{overtimehours}} </div>
                </div>
                <div class="top1_content">
                    <div style="display: flex;width: 50%;">
                        <div class="top2_title" style=" height: 80px;line-height: 80px; width: 50%;">
                            請假次數</div>
                        <div style=" width: 50%;">
                            <div class="top2_title" style="  width: 100%;">工作天</div>
                            <div class="top2_title" style="  width: 100%;">加班</div>
                        </div>
                    </div>
                    <div style="width: 50%;">
                        <div class="top2_content colorlinear_blue" style="width: 100%;"> {{offdays.toFixed(2)}} </div>
                        <div class="top2_content colorlinear_blue" style="width: 100%;"> {{overtimedays}} </div>
                    </div>
                </div>
            </div>
            <div class="top1">
                <div class="top1_title" @click="handle_score"> 成績單 </div>

                <div class="top1_content">
                    <div class="top2_title" style="height: 80px;line-height: 80px">加點 </div>
                    <div class="top2_content"
                        style="height: 80px;line-height: 80px;background:linear-gradient(to right,rgb(255, 182, 218), rgb(255, 222, 239)) ">
                        {{grade1}} </div>
                </div>
                <div class="top1_content">
                    <div class="top2_title" style="height: 80px;line-height: 80px"> 扣點 </div>
                    <div class="top2_content   colorlinear_green" style="height: 80px;line-height: 80px;"> {{grade0}}
                    </div>
                </div>
            </div>
        </div>
        <br>
        <a-card title="崗位技能培訓" size="small" v-bind:style="vis_card">
            <a-table class="basebg" :loading="skillloading" size="small" :bordered="true" :columns="columns"
                :pagination="false" :scroll="scroll" :data-source="skilltable" rowKey="id">
                <template v-for="(item,index) in stationlist" :slot="item.id" slot-scope="txt">
                    <div :key="index">
                        <a-icon v-if="txt" type="check-circle" theme="twoTone" two-tone-color="#52c41a" />
                        <a-icon v-else type="close-circle" theme="twoTone" two-tone-color="red" />
                    </div>
                </template>
            </a-table>
        </a-card>
    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'
    import { merge } from '@/utils/util'
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'

    const i18n = require('./i18n')
    // const formItems = [
    //     { key: 'worker', type: 'input', value: "2" },
    // ]

    export default {
        name: 'dashBoard',
        components: { EfMain },
        i18n: merge(require('@/i18n'), i18n),
        props: {
            dept: { type: String, default: "" },
        },
        data() {
            return {
                //  formItems,
                scorelist: [],
                stationlist: [],
                totalstation: 0,
                passstation: 0,
                skilltable: [],
                forms: { worker: "" },
                vis_card: { display: "none" },
                loadcount: 0,
                skillcount: 0,
                ondutydays: 0,
                overtimehours: 0,
                offdays: 0,
                overtimedays: 0,
                empno: "",
                grade1: 0,
                grade0: 0,
                name: "",
                first: "",
                second: "",
            };
        },
        watch: {

        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { x: this.stationlist.length * 100 + 100, y: this.pageMinHeight - 200 }
            },

            failstation() {
                return Number(this.totalstation) - Number(this.passstation)
            },
            loading() {
                return this.loadcount > 0;
            },
            skillloading() {
                return this.skillcount > 0;
            },
            columns() {
                var arr = [
                    { key: 'name', align: 'left', title: this.$t('name'), width: 100, fixed: 'left' },
                ];

                var temp = [];
                this.stationlist.map(e => {
                    temp.push({ key: e.id, title: e.name })
                })
                return [...arr, ...temp].map((e) => {
                    e.dataIndex = e.key
                    e.sorter = ['no', 'actions'].includes(e.key) ? false : (a, b) => this.sorter(a, b, e.key)
                    //e.slots = { title: this.$t(e.key) }
                    e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
                    return e
                })
            },


        },
        created() {
            this.handle_curdate();
            if (Object.values(this.$route.query).length > 0) {
                // this.formItems[0].value = this.$route.query.id;
                this.forms.worker = this.$route.query.id
                this.name=this.$route.query.name;
                this.handle_search(this.forms);
            }
            else {
                //  this.formItems[0].value = "";
            }
        },
        methods: {
            handle_curdate() {
                var mydate = new Date();
                var year = mydate.getFullYear();
                var month = mydate.getMonth() + 1;
                var second = year + '-' + month + '-25';
                var first = "";
                if (month == 1) {
                    first = (year - 1) + '-12-26';
                }
                else {
                    first = year + '-' + (month - 1) + '-26';
                }
                this.first = first;
                this.second = second
            },

            // submit(values) {
            //     this.forms = { ...values }
            //     this.handle_search(values);
            // },

            handle_search(values) {
                this.vis_card.display = "none";
                this.loadcount = 7;
                reqPost("WORKER", { prop: "searchscorecount", id: values.worker, idate: [this.first, this.second] }).then(res => {
                    //this.scorelist = res.data;
                    this.loadcount--;
                    res.data.some(e => {
                        if (e.grade == '1') {
                            this.grade1 = e.count
                        }
                        if (e.grade == '-1') {
                            this.grade0 = e.count
                        }
                    })
                })

                reqPost("WORKER", { prop: "stationcount", id: values.worker }).then(res => {
                    this.loadcount--;
                    this.totalstation = res.data[0].count;
                })

                reqPost("WORKER", { prop: "skillcount", id: values.worker }).then(res => {
                    this.loadcount--;
                    this.passstation = res.data[0].count;
                })

                var params = {
                    prop: "getstation",
                    worker: this.forms.worker
                }
                reqPost('WORKER', params).then(res => {
                    this.loadcount--;
                    this.stationlist = res.data;
                })

                reqPost("WORKER", { prop: "ondutydays", id: values.worker, org_id: "476", idate: [this.first, this.second] }).then(res => {
                    this.loadcount--;
                    this.ondutydays = res.data.data[0].count;
                    this.empno = res.data.empno;
                })


                reqPost("WORKER", { prop: "offdays", id: values.worker, org_id: "476", idate: [this.first, this.second] }).then(res => {
                    this.loadcount--;
                    this.offdays = 0;
                    res.data.map(e => {
                        this.offdays = this.offdays + Number(e.days)
                    })
                })

                reqPost("WORKER", { prop: "overtimedays", id: values.worker, org_id: "476", idate: [this.first, this.second] }).then(res => {
                    this.loadcount--;
                    this.overtimedays = res.data[0].count;
                })

                reqPost("WORKER", { prop: "getovertimes", id: values.worker, org_id: "476", idate: [this.first, this.second] }).then(res => {
                    this.loadcount--;
                    this.overtimehours = res.data[0].count || 0;
                })

                reqPost('WORKER', { prop: "getskill", worker: values.worker }).then(res => {
                    this.loadcount--;
                    this.skilltable = [{ id: values.worker }].map(e => {
                        this.stationlist.map(a => {
                            e[a.id] = false;
                            res.data.some(b => {
                                if (b.station == a.id) {
                                    e[a.id] = true
                                    e.name = b.name;
                                    this.name = b.name
                                    return true;
                                }
                            })

                        })
                        return e;
                    });
                })
            },

            handle_score() {
                this.$router.push("../../../sys/worker/"+this.$props.dept+"/scoremain?id=" + this.forms.worker + "&first=" + this.first + "&second=" + this.second)
                // window.open("../../../sys/worker/OQA/scoremain?id=" + this.forms.worker + "&first=" + this.first + "&second=" + this.second,"_self");
            },

            handle_erp() {
                this.$router.push("/sys/worker/"+this.$props.dept+"/dutyinfo?empno=" + this.empno + "&first=" + this.first + "&second=" + this.second + "&name=" + this.name)
                // window.open("../../../sys/worker/OQA/dutyinfo?empno=" + this.empno + "&first=" + this.first + "&second=" + this.second + "&name=" + this.name,"_self");
            },


            handle_skill() {
                if (this.vis_card.display == "none") {
                    this.vis_card.display = "block"

                }
                else if (this.vis_card.display == "block") {
                    this.vis_card.display = "none";
                }

            },
        },
    };
</script>

<style>
    @import url('./index.css');
</style>