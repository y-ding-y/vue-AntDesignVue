<template>
    <div class="user_skills" :class="'animated '+  animated" :style="styleSet">
        <div v-if="curdiv" :style="[styleclick]"> </div>
        <el-table :data="tableData" ref="table" height="100%" :row-style="rowstyle" :cell-style="cellstyle"
            :header-row-style="headerrowstyle" :header-cell-style="headercellstyle">
            <el-table-column v-for="(col,index) in columns" :key="index" :prop="col.prop" :label="col.label"
                :width="col.width" :align="col.align">
                <template slot-scope="scope">
                    <div :style="{color:col.color,fontWeight:col.fontweight}">
                        {{scope.row[col.prop]}}
                    </div>
                </template>
            </el-table-column>
        </el-table>
    </div>
</template>

<script>
    // import bus from './bus.js'
    const data = [{
        date: '1970-01-01',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1518 弄'
    }, {
        date: '1970-01-01',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1518 弄'
    },]
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
                result: data,
                refreshData: null,
                refreshpublic: null,
                scrolltable: null,
                busdata: [],
                animated: this.ss.inanimate,
                websocket: null,
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
                this.handle_search();
            },
            datesource() {
                this.handle_search();
            },
        },
        mounted() {
            this.tableScroll()
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
            clearInterval(this.refreshpublic);
            clearInterval(this.scrolltable);
            this.scrolltable = null
            this.refreshData = null;
            this.refreshpublic = null;
        },
        computed: {
            tableData() {
                return this.result
            },
            styleclick() {
                return {
                    border: this.curdiv ? '5px solid blue' : '0px',
                    position: 'absolute',
                    width: this.ss.width + "px",
                    height: this.ss.height + "px",
                }
            },
            styleSet() {
                var style = {
                    //  border: this.curdiv ? '5px solid blue' : '0px',
                    position: 'absolute',
                    top: this.ss.top + 'px',
                    left: this.ss.left + 'px',
                    width: this.ss.width + "px",
                    height: this.ss.height + "px",
                    fontSize: this.ss.fontSize + "px",
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
            rowstyle() {
                return {

                }
            },
            rowcell() {
                return this.ss.table.rowcell
            },
            headercell() {
                return this.ss.table.headercell
            },
            cellstyle() {
                return {
                    backgroundColor: this.rowcell.backgroundcolor,
                    color: this.rowcell.color,
                    fontSize: this.rowcell.fontsize + 'px',
                    border: "0px solid " + this.rowcell.backgroundcolor,
                    height: this.rowcell.height + "px",
                    padding: [0, 0, 0, 0],
                    fontWeight: this.rowcell.fontweight
                }
            },
            headerrowstyle() {
                return {

                }
            },
            headercellstyle() {
                return {
                    backgroundColor: this.headercell.backgroundcolor,
                    color: this.headercell.color,
                    fontSize: this.headercell.fontsize + 'px',
                    border: "0px solid " + this.headercell.backgroundcolor,
                    height: this.headercell.height + "px",
                    padding: [0, 0, 0, 0],
                    fontWeight: this.headercell.fontweight
                }
            },
            columns() {
                return this.ss.table.columns
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
        },
        methods: {
            tableScroll() {
                const table = this.$refs.table;
                // 拿到表格中承载数据的div元素
                const divData = table.bodyWrapper;
                // 拿到元素后，对元素进行定时增加距离顶部距离，实现滚动效果(此配置为每100毫秒移动1像素)
                this.scrolltable = setInterval(() => {
                    // 元素自增距离顶部1像素
                    divData.scrollTop += 2;
                    // 判断元素是否滚动到底部(可视高度+距离顶部=整个高度)
                    if (divData.clientHeight + divData.scrollTop == divData.scrollHeight) {
                        // 重置table距离顶部距离
                        divData.scrollTop = divData.scrollTop - divData.scrollHeight / 2;
                    }
                }, 30);  // 滚动速度
            },
            handle_search() {
                //let dtype = this.ss.datetype;
                let dsource = this.ss.datesource
                switch (dsource) {
                    case "phpapi":
                        this.handle_phpapi(dsource);
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
                        var temp = res.data
                        this.result = [...temp, ...temp]
                    })
                        .catch(error => {
                            console.log(error)
                            this.result = []
                            this.$message.error(this.ss.componentname + "NetWork Error")
                        })
                }
            },

            handle_publicdata(dsource, dtype) {
                let temp = eval("(" + this.ss[dsource] + ")")

                if (this.refreshData != null) {
                    this.$store.dispatch("tdpublic/refreshdata",
                        { key: Number(temp.key) }).then(() => {
                            publicdata = this.$store.getters['tdpublic/commondata']
                            var tempresult = this.$store.getters['tdpublic/commondata'][Number(temp.key)].tabledata
                            this.result = [...tempresult, ...tempresult]
                        })
                }
                else {
                    var publicdata = this.$store.getters['tdpublic/commondata']
                    if (publicdata.length > 0) {
                        if (dtype == "array") {
                            var tempresult = publicdata[Number(temp.key)].tabledata
                            this.result = [...tempresult, ...tempresult]
                        }
                    }
                }


                // let tempapi = eval("(" + publicdata[Number(temp.key)].note + ")")
                // this.$store.dispatch("tdpublic/refreshdata",
                //     { api: tempapi.api, params: tempapi.params, key: Number(temp.key) }).then(() => {

                //     }) 


                // let publicdatatab = Array(this.$localForage.getItem('3dpublicdata'))
                // Promise.all(publicdatatab).then(value => {
                //     if (value[0] != null) {
                //         if (value[0].length > 0) {
                //             let temp_public = value[0];
                //             if (dtype == 'string') {
                //                 var table = value[0][Number(temp.key)].tabledata
                //                 this.result = table[Number(temp.row) - 1][temp.colname]
                //             } else if (dtype == "array") {
                //                 //  this.result = value[0][Number(temp.key)].tabledata 
                //                 var tempresult = value[0][Number(temp.key)].tabledata
                //                 this.result = [...tempresult, ...tempresult]
                //             }

                //             if (temp.interval) {
                //                 this.refreshpublic = setInterval(() => {
                //                     let tempapi = eval("(" + value[0][Number(temp.key)].note + ")")
                //                     this.$funAxios.reqPost(tempapi.api, tempapi.params).then(res2 => {
                //                         temp_public[Number(temp.key)].tabledata = res2.data
                //                         this.$localForage.setItem('3dpublicdata', temp_public)

                //                         bus.$emit("publicdata", temp_public)

                //                     })
                //                 }, Number(temp.interval) * 1000 * 60);
                //             }
                //             else {
                //                 clearInterval(this.refreshpublic);
                //                 this.refreshpublic = null;
                //             }

                //         }
                //     }
                // })
            },

        }
    }
</script>


<style scoped>
    .user_skills /deep/ .el-table--scrollable-y .el-table__body-wrapper {
        overflow-y: hidden;
    }

    .user_skills {
        width: 50%;
        margin: auto;
    }

    .user_skills /deep/ .el-table--fit {
        padding: 20px;
    }

    .user_skills /deep/ .el-table,
    .el-table__expanded-cell {
        background-color: transparent;
    }

    .user_skills /deep/ .el-table tr {
        background-color: transparent;
    }

    .user_skills /deep/ .el-table th {
        background-color: transparent;
    }

    .user_skills /deep/ .el-table--enable-row-transition .el-table__body td,
    .el-table .cell {
        background-color: transparent;
    }

    .el-table::before {
        left: 0;
        bottom: 0;
        width: 100%;
        height: 0px;
    }
</style>
