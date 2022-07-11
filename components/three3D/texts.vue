<template>
    <div :class="' combtn animated '+ animated" :style="[styleSet,styleclick]">
        <div v-if="maincontent==null" class="loadingOne" :style="{height:styleSet.height+'px'}">
            <span></span>
            <span></span>
            <span></span>
            <span></span>
            <span></span>
        </div>

        {{maincontent}}
    </div>
</template>

<script>
    //  import merge from 'deepmerge'

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
        sockets: {
            // welcom: function (data) {
            //     console.log("welcome data 数据返回 = >", data);

            // },
        },
        data() {
            return {
                isshow: true,
                result: null,

                temp_nodejsemit: "",
                //maincontent: this.$props.ss.content,
                animated: this.ss.inanimate,
                refreshData: null,
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
                    this.result = this.ss.content
                }
            },
            datedetail() {
                this.handle_search();
            },
            datesource() {
                this.handle_search();
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
        computed: {
            maincontent() {
                if (this.ss.datebind) {
                    return this.result
                }
                else if (this.ss.content) {
                    return this.ss.content
                }
                else {
                    return ""
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

            styleSet: function () {
                var style = {
                    border: this.curdiv ? '5px solid blue' : '0px',
                    position: 'absolute',
                    top: this.ss.top + 'px',
                    left: this.ss.left + 'px',
                    width: this.ss.width + "px",
                    height: this.ss.height + "px",
                    fontSize: this.ss.fontSize + "px",
                    lineHeight: this.ss.height + "px",
                    zIndex: this.ss.zindex,
                    color: this.ss.color,
                    textAlign: this.ss.textAlign,
                    letterSpacing: this.ss.spacing + "px",
                    fontWeight: this.ss.fontweight,
                    textDecoration: this.ss.decoration,
                    fontStyle: this.ss.fontstyle,
                    //transform:this.ss.transform
                }
                //console.log(merge(this.ss, style))
                return style
            },
            styleclick() {
                return {
                    //   color: "red"
                }
            },
        },
        methods: {
            handle_search() {
                let dtype = this.ss.datetype;
                let dsource = this.ss.datesource
                switch (dsource) {
                    case "phpapi":
                        this.handle_phpapi(dsource, dtype);
                        break;
                    case "nodejsemit":
                        this.handle_nodejsemit(dsource, dtype);
                        break;
                    case "publicdata":
                        this.handle_publicdata(dsource, dtype);
                        break;
                    default:
                        break;
                }

            },

            handle_phpapi(dsource, dtype) {
                let temp = eval("(" + this.ss[dsource] + ")")
                if (temp.url != '') {
                    this.$funAxios.reqPost(temp.url, temp.params).then(res => {
                        // if (this.ss.dateshow == false) {
                        //     this.$store.commit("kanban/setTempdate", { kind: dtype, name: this.ss.componentname, val: res.data });
                        // }
                        if (typeof (res.data) == dtype) {
                            this.result = res.data
                        }
                        else {
                            this.$message.error(this.ss.componentname + "数据格式错误")
                        }
                    })
                        .catch(error => {
                            console.log(error)
                            this.result = null
                            this.$message.error(this.ss.componentname + "NetWork Error")
                        })
                }
            },

            handle_nodejsemit(dsource, dtype) {
                this.sockets.unsubscribe(this.temp_nodejsemit);
                this.sockets.subscribe(this.ss[dsource], data => {
                    this.temp_nodejsemit = this.ss[dsource]
                    if (typeof (data) == dtype) {
                        this.result = data
                    }
                    else {
                        this.$message.error(this.ss.componentname + "数据格式错误")
                    }
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
                                console.log(this.result)
                            }
                        }
                    }
                })
            },


        },
    }
</script>

<style scoped>
    .title {
        width: 100%;
        text-align: center;
        margin: 60px 0;
        font-size: 18px;
        color: #999;
    }

    .loadingOne {
        width: 80px;
        margin: 0 auto;
    }

    .loadingOne span {
        display: inline-block;
        width: 8px;
        height: 100%;
        border-radius: 4px;
        background: lightgreen;
        -webkit-animation: load 1s ease infinite;
        animation: load 1s ease infinite;
    }

    @-webkit-keyframes load {

        0%,
        100% {
            height: 20px;
            background: lightgreen;
        }

        50% {
            height: 30px;
            margin: -15px 0;
            background: lightblue;
        }
    }

    .loadingOne span:nth-child(2) {
        -webkit-animation-delay: 0.2s;
        animation-delay: 0.2s;
    }

    .loadingOne span:nth-child(3) {
        -webkit-animation-delay: 0.4s;
        animation-delay: 0.4s;
    }

    .loadingOne span:nth-child(4) {
        -webkit-animation-delay: 0.6s;
        animation-delay: 0.6s;
    }

    .loadingOne span:nth-child(5) {
        -webkit-animation-delay: 0.8s;
        animation-delay: 0.8s;
    }
</style>
