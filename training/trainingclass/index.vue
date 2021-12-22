<template> 
    <ef-main :isMainPage="true" :loading="loading" :title="'排課'">
        <a-form :label-col="{ span: 5 }" :wrapper-col="{ span: 16 }">
            <a-form-item>
                <template slot="label">
                    <span style="color:red">*</span> <b>{{$t('name')}}</b>
                </template>
                <a-select v-model="course" show-search :filter-option="filterOption">
                    <a-select-option v-for="(item,index) in courselist" :value="item.id" :key="index"
                        :title="item.coursename">
                        {{item.coursename}}
                    </a-select-option>
                </a-select>
            </a-form-item>
            <a-form-item>
                <template slot="label">
                    <span style="color:red">*</span> <b>{{$t('place')}}</b>
                </template>
                <a-input v-model="place" style="width: 100%" :placeholder="$t('place')" />
            </a-form-item>
            <a-form-item>
                <template slot="label">
                    <span style="color:red">*</span> <b>{{$t('daterange')}}</b>
                </template>
                <a-range-picker :key=" formi" :show-time="{ format: 'HH:mm' }" format="YYYY-MM-DD HH:mm"
                    :placeholder="['Start Date', 'End Date']" @change="onChange" v-model="daterange" />
            </a-form-item>
            <a-form-item>
                <template slot="label">
                    <b>{{$t('teacher')}}</b>
                </template>
                <a-input v-model="teacher" style="width: 100%" :placeholder="$t('teacher')" />
            </a-form-item>
            <!-- <a-form-item>
                <template slot="label">
                    <span style="color:red">*</span><b>{{$t('student')}}</b>
                </template>

                <a-textarea v-if="!empnos.length>0" disabled></a-textarea>
                <div v-else :key="i"
                    style="border: 1px solid #d9d9d9;border-radius: 4px;padding: 4px 11px;background-color: #f5f5f5;">
                    <a-tag v-for='(item,index) in empnos' closable @close="handle_close(index)" :key="'i'+index">
                        {{item.name}}({{item.empno}})</a-tag>
                </div>
                <a-spin :spinning="spinloading">
                    <a-input v-model.trim="student" style="width: 120px" placeholder="工號"
                        @keyup.enter="handle_search" />
                    <a-button style="margin-left: 10px;" :class="'success'" shape="circle" icon="plus" size="small"
                        @click="handle_search">
                    </a-button>
                </a-spin>
            </a-form-item> -->
            <a-form-item>
                <template slot="label">
                    <b>{{$t('note')}}</b>
                </template>
                <a-textarea v-model="note" :rows="3" :placeholder="$t('note')" />
            </a-form-item>
        </a-form>

        <a-button :class="'success'" @click="handle_add()" style="width:100%">
            <a-icon type="check" /> OK
        </a-button>



    </ef-main>
</template>
<script>
    import { reqPost } from '@/services/base'
    import EfMain from "@/components/page/main";
    import { merge, } from '@/utils/util'
    import { mapState } from 'vuex'
    import moment from 'moment'
    import bus from './../bus.js'
    //厂别，投诉日期，投诉类别，投诉人，机种，问题描述，原因类别，上传文件

    const i18n = require('./i18n')
    export default {
        name: 'trainingdept',
        components: { EfMain },
        i18n: merge(require('@/i18n'), i18n),
        props: {
            dept: { type: String, default: "" },
            params: Object,
            actionstatus: { type: String, default: "" }
        },
        data() {
            return {
                loadcount: 0,
                id: "",
                deptid: "",
                name: "",
                note: "",
                course: "",
                teacher: "",
                student: "",
                place: "",
                values: {},
                courselist: [],
                empnos: [],
                spinloadcount: 0,
                i: 0,
                formi: 0,
                datefm: null,
                dateto: null,
            };
        },
        watch: {
            empnos() {
                this.i++;
            }
        },
        computed: {
            daterange: {
                get() {
                    if (this.datefm == null) {
                        return [null, null]
                    }
                    else {
                        return [moment(this.datefm, 'YYYY-MM-DD HH:mm'), moment(this.dateto, 'YYYY-MM-DD HH:mm')];
                    }
                },
                set(v) {
                    console.log(v)
                    return v;
                }


            },
            icons() {
                return process.env.VUE_APP_API_WEB_URL;
            },
            fileapi() {
                return process.env.VUE_APP_API_BASE_URL
            },
            scroll() {
                return { x: 10, y: this.pageMinHeight - 200 }
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),

            loading() {
                return this.loadcount > 0;
            },
            spinloading() {
                return this.spinloadcount > 0;
            },
        },
        created() {
            this.loadcount = 2;
            reqPost('TRAINING', { prop: "getdept", dept: this.$props.dept }).then(res => {
                this.loadcount--;
                this.deptid = res.data[0].id;
                reqPost('TRAINING', { prop: "getcourse", dept: this.deptid }).then(res => {
                    this.loadcount--;
                    this.courselist = res.data;
                    console.log(this.params)
                    if (this.actionstatus == "updateclass") {
                        this.course = this.params.course;
                        this.place = this.params.place;
                        this.datefm = this.params.date_from;
                        this.dateto = this.params.date_to;
                        this.teacher = this.params.teacher;
                        this.note = this.params.note;
                    }
                })
            })


        },
        methods: {
            filterOption(input, option) {
                return (
                    option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
                );
            },
            moment,
            onChange(value, dateString) {
                this.datefm = dateString[0] || null;
                this.dateto = dateString[1] || null;
                console.log(this.datefm)
            },

            handle_add() {
                console.log(this.datefm)
                var name = "";
                this.courselist.some(e => {
                    if (e.id == this.course) {
                        name = e.coursename
                        return true
                    }
                })
                var params = {
                    prop: this.actionstatus == "updateclass" ? "updateclass" : "insertclass",
                    course: this.course,
                    coursename: name,
                    place: this.place,
                    datefm: this.datefm,
                    dateto: this.dateto,
                    // empnos: this.empnos.map(e => {
                    //     return e.empno
                    // }),
                    teacher: this.teacher,
                    note: this.note,
                    dept: this.deptid,
                    id: this.actionstatus == "updateclass" ? this.params.classid : ""
                }
                console.log(params)
                if (this.datefm.slice(0, 10) != this.dateto.slice(0, 10)) {
                    this.$message.error("起止日期不在同一天");
                }
                else {
                    if (this.course != "" && this.place != "" && this.datefm != null
                        && this.dateto != null) {
                        this.loadcount = 1;
                        reqPost('TRAINING', params).then(res => {
                            this.loadcount--;
                            if (res.data > 0) {
                                this.$message.success((this.actionstatus == "updateclass" ? "Update" : "Add ") + " Success");
                                this.handle_clear();
                                bus.$emit("trainingclass", {
                                    _visible: false,
                                    status: "ok"
                                })
                            }
                            else {
                                this.$message.error((this.actionstatus == "updateclass" ? "Update" : "Add ") + " Error");
                            }
                        })
                    }
                    else {
                        this.$message.error("有必填項目為空")
                    }
                }

            },
            handle_clear() {
                this.course = "";
                this.dateto = null;
                this.datefm = null;
                this.empnos = [];
                this.student = "";
                this.teacher = "";
                this.note = "";
                this.place = "";
                this.formi++;
            },
            handle_search() {
                if (this.student != "") {
                    var flag = 1;
                    if (this.empnos.length > 0) {
                        this.empnos.some(e => {
                            if (e.empno.toUpperCase() == this.student.toUpperCase()) {
                                flag = 0;
                                return true;
                            }
                        })
                    }

                    if (flag != 0) {
                        this.spinloadcount = 1;
                        reqPost("WORKER", { prop: "geterpempno", empno: this.student }).then(res => {
                            this.spinloadcount--;
                            // temp.stationlist = this.handle_skillstation(e); 
                            if (res.data.length > 0) {
                                this.empnos.push({
                                    empno: res.data[0].empno,
                                    name: res.data[0].name,
                                })
                                this.student = "";
                            }
                            else {
                                this.$message.warning("工號不存在")
                            }
                            //   this.tabledata.splice(index, 1, temp);
                        })
                    } else {
                        this.$message.error("該學員已添加")
                    }
                }
                else {
                    this.$message.error("請輸入工號")
                }

            },

            handle_close(e) {
                this.empnos.splice(e, 1)
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
        color: white;
        background-color: @success-color;
    }

    .warning,
    .warning:hover,
    .warning:visited {
        /* background-color: @success-color; */
        color: @warning-color;
    }

    .img {
        width: 2rem;
        height: 2rem;
        border-radius: .04rem;
        margin-left: .3rem;
        margin-top: .3rem;
        object-fit: cover;
    }
</style>