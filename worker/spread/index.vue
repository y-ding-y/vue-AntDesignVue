<template>
    <ef-main :isMainPage="true" :loading="loading" :key="id">
        <div style="margin: 5px; ">
            <a-button type="primary" @click="handle_toannuchart">年資看板</a-button>
            &nbsp;&nbsp;&nbsp;
            <a-button type="primary" v-if="stationcount" ghost @click="handle_toscoredash('searchallscore')">成績單一覽表
            </a-button>
            &nbsp;&nbsp;&nbsp;
            <a-button type="primary" v-if="stationcount" ghost @click="handle_toscoredash('searchallskill')">技能一覽表
            </a-button>
            &nbsp;&nbsp;&nbsp;
            <a-button type="primary" v-if="stationcount" ghost @click="handle_toscoredash('searchallannu')">出勤一覽表
            </a-button>
            &nbsp;&nbsp;&nbsp;
            標註：
            <a-switch default-checked v-model="canBiaoZhu" />
        </div>
        <div style="margin: 5px;  display: flex;  text-align: center;font-size: 16px;font-weight: 700;">
            <div
                style="line-height: 30px;height: 30px; margin: 5px; width: 15%;background-color: rgb(205, 252, 255);border: 1px solid;">
                正式工：{{per1}}
            </div>
            <div
                style="line-height: 30px;height: 30px;margin: 5px;width: 15%;background-color: rgb(255, 154, 230) ;border: 1px solid;">
                試用期：{{per2}}
            </div>
            <div
                style="line-height: 30px;height: 30px;margin: 5px;width: 15%;background-color: rgb(236, 117, 127);border: 1px solid;">
                臨時工：{{per3}}
            </div>
        </div>

        <div class="myBiaoZhu" id="myBiaoZhuDiv">
            <img id="myBiaoZhu" :src="srcurl" style="display:block; width:100%; max-width:100%; height:100%;">

            <!-- <img id="myBiaoZhu" src="./test.jpg" style="display:block; width:100%; max-width:100%; height:auto;"> -->

            <!-- <a-button type="text" @click="startBiaoZhu">开始标注</a-button>
            <a-button type="text" @click="endBiaoZhu">标注完成</a-button> -->
        </div>

        <div style="display: flex;justify-content: right ">
            <div style="width: 70%; display: flex;flex-wrap: wrap;justify-content: right; ">
                <div class="offline" @click="handle_toinfo(item.userid,item.username)"
                    style="width:150px; border: 1px solid;text-align: center;background-color: rgb(205, 252, 255);"
                    v-for="(item,index) in offstation" :key="index">
                    <div style="font-size: 16px;font-weight: 700;color: black;">{{item.name}}</div>
                    <div style="font-size: 14px;font-weight: 600;color: rgb(255, 0, 0);" v-if="!item.username">无人员安排
                    </div>
                    <div style="font-size: 14px;font-weight: 600;color: black;" v-else>{{item.username}}</div>
                </div>
            </div>
        </div>
        <a-modal title="Add Person" :visible="visible" :footer="null" @cancel="handleCancel">
            <a-form :label-col="{ span: 8 }" :wrapper-col="{ span: 10 }">
                <upload-form v-if="onstation.length>0" :istitile="false" :isMainPage="false" :form-items="formItems"
                    @submit="handle_layout" :key="i">

                </upload-form>
            </a-form>
        </a-modal>
    </ef-main>
</template>

<script>
    // import G6 from '@antv/g6';
    import { reqPost } from '@/services/base'
    import EfMain from '@/components/page/main'
    import { merge } from '@/utils/util'
    import UploadForm from '@/components/page/uploadform'
    import { mapState } from 'vuex'

    export default {
        name: 'workerSpread',
        i18n: merge(require('@/i18n'), require('./i18n')),
        components: { EfMain, UploadForm },
        props: {
            dept: { type: String, default: "" },
        },
        data() {
            return {
                banMa: [],           //斑马线的数组
                visible: false,
                canBiaoZhu: false,  //是否可以进行标注
                pointColor: 'red',   //点的颜色
                pointSize: 10,       //点的大小
                x: 0,
                y: 0,
                formItems: [],
                loadcount: 0,
                deptlist: [],
                deptid: "",
                deptname: "",
                imgid: "",
                stationcount: 0,
                stationlist: [],
                onstation: [],
                temp_onstation: [],
                offstation: [],
                i: 0,
                img_width: 0,
                img_height: 0,
                workers: [],

                worker1: 0,//正式工
                worker2: 0,//試用期
                worker3: 0,//臨時工

                first: "",
                second: "",
                id: 0,
            }
        },
        created() {
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
            window.handle_toinfo = this.handle_toinfo; //
            this.loadcount = 1;
            reqPost('WORKER', { prop: "getdeptid", name: this.$props.dept }).then(res => {
                this.loadcount--;
                this.deptid = res.data;
                this.handle_searchworker();
                this.handle_searchImg();

            })
        },
        watch: {
            dept() {
                this.id++;
            },
            img_height() {
                this.banMa.map(e => {
                    if (e.station) {
                        var div = document.getElementById(e.station);
                        document.getElementById('myBiaoZhuDiv').removeChild(div)

                        this.createMarker(this.img_width * Number(e.x).toFixed(6),
                            this.img_height * Number(e.y).toFixed(6), e.station, e.x_offset, e.y_offset)
                    }
                })
            }
        },
        computed: {
            srcurl() {
                return this.imgid != "" ? (this.icons + '/file/' + this.imgid + '/large') : ""
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),
            icons() {
                return process.env.VUE_APP_API_WEB_URL;
            },
            loading() {
                return this.loadcount > 0;
            },
            totalworker() {
                return this.workers.length
            },
            per1() { return ((this.worker1 / this.totalworker) * 100).toFixed(2) + "%" },
            per2() { return ((this.worker2 / this.totalworker) * 100).toFixed(2) + "%" },
            per3() { return ((this.worker3 / this.totalworker) * 100).toFixed(2) + "%" },
        },

        methods: {
            submit(values) {
                this.deptid = values.dept;
                this.handle_searchImg();
            },

            handle_searchImg() {
                reqPost("WORKER", { prop: "getdept", id: this.deptid }).then(res => {
                    this.imgid = res.data[0].layout
                    this.handle_searchstation();
                })
            },
            handle_searchworker() {
                this.loadcount = 1;
                var params = {
                    prop: "getworker",
                    id: this.deptid
                }
                reqPost('WORKER', params).then(res => {
                    this.loadcount--;
                    this.worker1 = 0; this.worker2 = 0; this.worker3 = 0;
                    this.workers = res.data.map(e => {
                        e.annu = Number(e.annu)
                        if (e.salary_type == "1") {
                            if (e.annu < 0.25) {
                                this.worker2++;
                            }
                            else {
                                this.worker1++;
                            }
                        }
                        else if (e.salary_type == "6") {
                            this.worker3++;
                        }
                        return e;
                    })
                    // reqPost('WORKER', { prop: "getannuchart", detp: this.deptid }).then(data => {
                    //     var model = data.data;
                    //     this.workers = res.data.map(e => {
                    //         model.some(a => {
                    //             if (a.empno == e.empno) {
                    //                 e.annu = Number(a.annu)
                    //                 return true;
                    //             }
                    //         })
                    //         return e;
                    //     })
                    //     console.log(this.workers)
                    // })

                })
            },
            handle_searchstation() {
                this.img_width = document.querySelector("#myBiaoZhu").clientWidth
                setTimeout(() => {
                    this.img_height = document.querySelector("#myBiaoZhu").clientHeight
                    reqPost("WORKER", { prop: "getstation", id: this.deptid }).then(res => {
                        this.stationcount = res.data.length;
                        this.stationlist = res.data
                        res.data.length > 0 && res.data.map(e => {
                            if (e.type == 'online') {
                                if (e.x_per != null) {
                                    this.temp_onstation.push({ key: e.id, title: e.name, value: e.id })
                                    this.banMa.push({
                                        id: this.banMa.length + 1,
                                        x: Number(e.x_per),
                                        y: Number(e.y_per),
                                        station: e.id,
                                        x_offset: Number(e.x_offset),
                                        y_offset: Number(e.y_offset)
                                    })
                                    this.canBiaoZhu = true
                                    this.createMarker(this.img_width * Number(e.x_per).toFixed(6),
                                        this.img_height * Number(e.y_per).toFixed(6), e.id, Number(e.x_offset), Number(e.y_offset))
                                }
                                else {
                                    this.onstation.push({ key: e.id, title: e.name, value: e.id })
                                }

                            }
                            else {
                                this.workers.some(o => {
                                    if (o.station == e.id) {
                                        e.username = o.name;
                                        e.userid = o.id;
                                        return true
                                    }
                                })
                                this.offstation.push({ ...e })
                            }
                        })
                        this.canBiaoZhu = false

                    })
                }, 1000);
            },

            handle_layout(values) {
                if (values.station == "") {
                    this.$message.warning("請選擇工站");
                }
                else {
                    var params = {
                        prop: "updatestationXY",
                        x_per: values.x_per,
                        y_per: values.y_per,
                        station: values.station,
                        X_OFFSET: values.left,
                        Y_OFFSET: values.top,
                    };
                    reqPost("WORKER", params).then(res => {
                        if (res.data > 0) {
                            var index = this.onstation.findIndex(function (value) {
                                return value.key == values.station
                            })
                            this.temp_onstation.push(this.onstation[index])
                            this.onstation.splice(index, 1);
                            this.banMa.push({
                                id: values.station,
                                x: Number(values.x_per),
                                y: Number(values.y_per),
                                station: values.station,
                                x_offset: Number(values.left),
                                y_offset: Number(values.top)
                            })
                            this.createMarker(this.x, this.y, values.station, values.left, values.top)
                            this.$message.success("Add Success")
                            this.visible = false;
                        }
                        else {
                            this.$message.error("Add Error")
                        }
                    })
                }
            },
            handleCancel() {
                this.visible = false;
            },

            //开始标注
            startBiaoZhu() {
                this.canBiaoZhu = true
            },
            //完成标注
            endBiaoZhu() {
                this.canBiaoZhu = false
            },
            //画点
            createMarker(x, y, station, X_OFFSET, Y_OFFSET) {
                var div = document.createElement('div')
                div.className = 'marker'
                //  div.id = 'marker' + this.banMa.length
                div.id = station
                y = y + document.getElementById('myBiaoZhu').offsetTop - this.pointSize / 2
                x = x + document.getElementById('myBiaoZhu').offsetLeft - this.pointSize / 2
                div.style.width = this.pointSize + 'px'
                div.style.height = this.pointSize + 'px'
                div.style.backgroundColor = this.pointColor
                div.style.left = x + 'px'
                div.style.top = y + 'px'
                div.oncontextmenu = ((e) => {
                    var id = e.target.id
                    this.handle_delete(id)
                    // console.log(deleteres)
                    // if (deleteres == "1") {
                    //     document.getElementById('myBiaoZhuDiv').removeChild(div)
                    //     this.banMa = this.banMa.filter(item => item.id != id.slice(6, id.length))
                    // }

                    // if (e && e.preventDefault) {
                    //     //阻止默认浏览器动作(W3C)
                    //     e.preventDefault()
                    // } else {
                    //     //IE中阻止函数器默认动作的方式
                    //     window.event.returnValue = false
                    // }
                    return false
                })  //阻止冒泡行为和默认右键菜单事件，删除该点
                var str = "";
                this.workers.map(e => {
                    if (e.station == station) {
                        var strstyle = "";
                        if (e.salary_type == "1") {
                            if (e.annu < 0.25) {
                                strstyle = "style='background-color:rgb(255, 154, 230) '"
                            }
                            else {
                                strstyle = "style='background-color:rgb(205, 252, 255)'"
                            }
                        }
                        else if (e.salary_type == "6") {
                            strstyle = "style='background-color:rgb(236, 117, 127) '"
                        }

                        str = str + "<button " + strstyle + " class='ant-btn ant-btn-text' onclick=\"handle_toinfo(" + e.id + ",'" + e.name + "')\">" + e.name + "</button>"
                    }
                })
                if (str == "") {
                    str = "<b>无安排人员</b>"
                }

                div.innerHTML = " <div style='position: relative;top:" + Y_OFFSET + "px;width:80px;right:" + X_OFFSET + "px;display:flex;justify-content:spance-around;flex-direction:column' >" + str + "</div>"
                document.getElementById('myBiaoZhuDiv').appendChild(div)
            },

            delete_point(station) {
                var div = document.getElementById(station);
                document.getElementById('myBiaoZhuDiv').removeChild(div)
                this.banMa = this.banMa.filter(item => item.station != station)

            },

            handle_delete(e) {
                var params = {
                    prop: "deletestationXY",
                    station: e,

                };
                if (this.canBiaoZhu) {
                    reqPost("WORKER", params).then(res => {
                        if (res.data > 0) {
                            var index = this.temp_onstation.findIndex(function (value) {
                                return value.key == e
                            })
                            this.onstation.push(this.temp_onstation[index])
                            this.temp_onstation.splice(index, 1);
                            this.$message.success("Delete Success")
                            this.delete_point(e);
                        }
                        else {
                            this.$message.error("Add Error")
                        }
                    })
                }
                else {
                    this.$message.warning("請打開標註")
                }
            },
            handle_toinfo(e, name) {
                if (e) {
                    this.$router.push("../../../sys/worker/" + this.$props.dept + "/dashboard?id=" + e + "&name=" + name)
                    // window.open("../../../sys/worker/OQA/dashboard?id=" + e,"_self");
                }
            },
            handle_toannuchart() {
                this.$router.push("../../../sys/worker/" + this.$props.dept + "/annuchart")
                // window.open("../../../sys/worker/OQA/annuchart","_self");
            },
            handle_toscoredash(e) {
                this.$router.push("../../../sys/worker/" + this.$props.dept + "/scoredash?first=" + this.first + "&second="
                    + this.second + "&prop=" + e
                    + (this.stationcount > 0 ? ("&stationcount=" + this.stationcount) : "")
                )
                //  this.$router.push({name:"一覽表",params:{station:this.stationlist,first:this.first,second:this.second,prop:e,stationcount:this.stationcount}})
            },
        },
        destroyed() {
            window.onresize = null
        },
        mounted() {
            const that = this
            window.onresize = () => {

                return (() => {
                    that.img_width = document.querySelector("#myBiaoZhu").clientWidth
                    that.img_height = document.querySelector("#myBiaoZhu").clientHeight
                })()
            }

            document.getElementById('myBiaoZhu').oncontextmenu = ((e) => {
                if (e && e.preventDefault) {
                    //阻止默认浏览器动作(W3C)
                    e.preventDefault()
                } else {
                    //IE中阻止函数器默认动作的方式
                    window.event.returnValue = false
                }
                return false
            })      //阻止冒泡行为和默认右键菜单事件
            document.getElementById('myBiaoZhu').onmousedown = (e) => {
                e = e || window.event
                if (e.button !== 2) {       //判断是否右击
                    if (this.canBiaoZhu) {    //判断是否可以进行标注
                        var x = e.offsetX || e.layerX
                        var y = e.offsetY || e.layerY
                        console.log(x, y)
                        var myImg = document.querySelector("#myBiaoZhu")
                        var currWidth = myImg.clientWidth
                        var currHeight = myImg.clientHeight
                        var ProportionWidthInImg = x / currWidth
                        var ProportionHeightInImg = y / currHeight
                        // var ProportionWidthInImg = x / this.img_width
                        // var ProportionHeightInImg = y / this.img_height
                        // console.log("图片比例高度：" + ProportionHeightInImg)
                        // console.log("图片比例宽度：" + ProportionWidthInImg)
                        // console.log(this.banMa)
                        this.banMa.push({
                            id: this.banMa.length + 1,
                            x,
                            y
                        })
                        this.x = x;
                        this.y = y;
                        this.visible = true;
                        this.i++;
                        this.formItems = [
                            { key: 'station', type: 'select', options: this.onstation, span: "3" },
                            { key: 'x_per', type: 'input', value: ProportionWidthInImg, span: "3", disabled: true },
                            { key: 'y_per', type: 'input', value: ProportionHeightInImg, span: "3", disabled: true },
                            { key: 'top', type: 'numberinput', value: 10, span: "3", },
                            { key: 'left', type: 'numberinput', value: 40, span: "3", },
                        ]
                    }
                }
            }
        },
    }


</script>

<style lang="less">
    #myBiaoZhuDiv {
        position: relative;

        img {
            border: solid 1px #000;
            display: inline-block;
            z-index: 1;
        }

        .marker {
            position: absolute;
            border-radius: 50%;
            z-index: 999;

        }
    }

    .offline:hover {
        cursor: pointer;
        background-color: rgb(210, 250, 255);
    }
</style>