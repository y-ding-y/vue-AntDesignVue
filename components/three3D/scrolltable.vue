<template>
    <div :class="'animated '+  animated" :style="styleSet">
        <div style="display: flex;">
            <div v-for="(item,i) in tablecolum" :key="i"
                :style="{fontSize:styleSet.fontSize+'px',width:item.width+'%',marginLeft:'4px'}">
                {{item.name}}
            </div>
        </div>

        <vue-seamless-scroll :style="scrollStyle " style="height: 400px" :data="result" class="seamless-warp"
            :class-option="defaultOption">
            <ul id="scrollList" style="list-style: none;padding: 0;
            margin: 0 auto;">
                <li id="li1" v-for="(item,index) in result" :key="index" style="display: flex;"
                    :style="{height:ss.height+'px',lineHeight:ss.height+'px'}">

                    <div v-for="(k,i) in tablecolum" :key="i"
                        :style="{fontSize:styleSet.fontSize+'px',width:k.width+'%',marginLeft:'4px'}">
                        {{item[k.key]}}
                    </div>
                </li>
            </ul>
        </vue-seamless-scroll>
    </div>
</template>

<script>
    export default {
        props: {
            ss: {
                type: Object, default: () => ({
                    left: 10,
                    top: 10,
                    width: 100,
                    height: 100,
                    fontSize: 20
                })
            },
            curdiv: { type: Boolean, default: false }
        },
        data() {
            return {
                isshow: true,
                i: 0,
                modelIo: [],
                result: [],
                temp_nodejsemit: "",
                animated: this.ss.inanimate,
                refreshData:null,
            }
        },
        watch: {
            isshow(vnew, vold) {
                if (this.isshow) {
                    this.animated = this.ss.inanimate
                }
                else if (vnew != vold && !this.isshow) {
                    this.animated = this.ss.outanimate
                }
            },
            datebind(vnew, vold) {
                if (this.datebind && vnew != vold) {
                    this.handle_search();
                }
                else {
                    this.result = []
                }
            },
            datarefresh() {
                if (this.datarefresh) {
                    this.refreshData = setInterval(() => {
                        this.handle_search()
                    }, Number(this.ss.refreshdatatime) * 1000 * 60);
                } else {
                    clearInterval(this.refreshData);
                    this.refreshData = null;
                }

            },
            datedetail() {
                console.log("1")
                this.handle_search();
            },
            datesource() {
                this.handle_search();
            },
            tablecolum() {
                this.i++
            },
            styleSet() {
                this.i++
            },
            scrollStyle() {
                this.i++
            },
        },
        computed: {
            tablecolum() {
                return this.ss.scrolltablecols ?? []
            },
            maincontent() {
                if (this.ss.dateshow) {
                    return this.result
                }
                else {
                    return this.ss.content
                }
            },
            datarefresh() { 
                return this.ss.refreshdata
            },
            datebind() {
                return this.ss.datebind
            },
            datesource() {
                return this.ss.datesource
            },
            datedetail() {
                return this.ss[this.ss.datesource]
            },

            defaultOption() {
                return {
                    step: 1, //数值越大速度滚动越快
                    limitMoveNum: 6, //开始无缝滚动的数据量  //this.fourDatata.length
                    hoverStop: true, //是否开启鼠标悬停stop
                    direction: 1, // 0向下 1向上 2向左 3向右
                    openWatch: true, //开启数据实时监控刷新dom
                    singleHeight: 0, //单步运动停止的高度(默认值0是无缝不停止的滚动) direction => 0/1
                    singleWidth: 0, //单步运动停止的宽度(默认值0是无缝不停止的滚动) direction => 2/3
                    waitTime: 1000 //单步运动停止的时间(默认值1000ms)

                }
            },
            styleSet() {
                var style = {
                    border: this.curdiv ? '5px solid blue' : '0px',
                    position: 'absolute',
                    top: this.ss.top + 'px',
                    left: this.ss.left + 'px',
                    width: this.ss.width + "px",
                    height: this.result.length == 0 ? (this.ss.height + "px") : ((7 * this.ss.height) + 'px'),
                    fontSize: this.ss.fontSize + "px",
                    //lineHeight: this.ss.height + "px",
                    zIndex: this.ss.zindex,
                    color: this.ss.color,
                    textAlign: this.ss.textAlign,
                    letterSpacing: this.ss.spacing + "px",
                    fontWeight: this.ss.fontweight,
                    textDecoration: this.ss.decoration,
                    fontStyle: this.ss.fontstyle,
                }
                return style

            },

            scrollStyle() {
                var style = {
                    height: this.result.length == 0 ? (this.ss.height + "px") : ((6 * this.ss.height) + 'px'),
                    fontSize: this.ss.fontSize + "px",
                    //lineHeight: this.ss.height + "px", 
                    color: this.ss.color,
                    textAlign: this.ss.textAlign,
                    letterSpacing: this.ss.spacing + "px",
                    fontWeight: this.ss.fontweight,
                    textDecoration: this.ss.decoration,
                    fontStyle: this.ss.fontstyle,
                }
                return style
            },
        },
        created() {
            if (this.datebind) {
                if (this.datarefresh) {
                    this.refreshData = setInterval(() => {
                        this.handle_search()
                    }, Number(this.ss.refreshdatatime) * 1000 * 60);
                }
                else {
                    this.handle_search()
                }
            }
            this.isshow = this.ss.isshow
            if (this.ss.isshow) {
                this.animated = this.ss.inanimate
            }
            else {
                this.animated = this.ss.outanimate
            }
        },
        beforeDestroy() {
            clearInterval(this.refreshData);
            this.refreshData = null;
        },
        methods: {
            handle_search() {
                //let dtype = this.ss.datetype;
                let dsource = this.ss.datesource
                switch (dsource) {
                    case "phpapi":
                        this.handle_phpapi(dsource);
                        break;
                    case "nodejsemit":
                        this.handle_nodejsemit(dsource);
                        break;
                    case "publicdata":
                        this.handle_publicdata(dsource, this.ss.datetype);
                        break;
                    default:
                        break;
                }

            },

            handle_phpapi(dsource) {
                let temp = eval("(" + this.ss[dsource] + ")")
                if (temp.url != '') {
                    this.$funAxios.reqPost(temp.url, temp.params).then(res => {
                        this.result = res.data
                        this.i++
                    })
                        .catch(error => {
                            console.log(error)
                            this.result = []
                            this.$message.error(this.ss.componentname + "NetWork Error")
                        })
                }
            },

            handle_nodejsemit(dsource) {
                this.sockets.unsubscribe(this.temp_nodejsemit);
                this.sockets.subscribe(this.ss[dsource], data => {
                    this.temp_nodejsemit = this.ss[dsource]
                    this.result = data

                })
            },
            handle_publicdata(dsource, dtype) {
                let temp = eval("(" + this.ss[dsource] + ")")
                let publicdatatab = Array(this.$localForage.getItem('3dpublicdata'))
                Promise.all(publicdatatab).then(value => {
                    if (value[0] != null) {
                        if (value[0].length > 0) {
                            if (dtype == 'string') {
                                var table = value[0][Number(temp.key)].tabledata
                                this.result = table[Number(temp.row) - 1][temp.colname]
                            } else if (dtype == "array") {
                                this.result = value[0][Number(temp.key)].tabledata
                            }
                        }
                    }
                })
            },

        },
    }
</script>
