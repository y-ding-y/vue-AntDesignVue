<template>
    <ef-main :isMainPage="true" :uploadjson="true" @uploadjson="handle_selectjson" :fullscreen="true"
        @fullscreen="fullScreen()" :group="true" @group="groupedit">

        <!-- <a-button type="primary" @click="fullScreen()" size="small" :disabled="dd.length==0">全屏</a-button> -->
        <!-- <a-upload name="file" :multiple="false" @change="handle_selectjson" :file-list="fileList" :action="uploadapi">
            <a-button :class="'success'" size="small">上传样式</a-button>
        </a-upload> -->
        <a-spin :spinning="loading">
            <div class="maincenter" id="maincenter" :style="{height: (pageMinHeight-140)+'px'}">
                <!-- class="fullscreen" -->
                <div style="height: 100%" id="fulls" ref="fulls">
                    <div id="screen"
                        :style="{'width':style.width+'px','height':style.height+'px','transform': style.transform }">
                        <div v-for="(item,index) in dd" :key="index">
                            <component :ref="item.name+item.key" :vwidth="style.width" :vheight="style.height"
                                v-styleclick="{refname:item.name+item.key,actions:item.ss.actions,isshow:item.ss.isshow}"
                                :ss="item.ss" :is="item.com">
                            </component>
                        </div>
                    </div>
                </div>

            </div>
        </a-spin>

    </ef-main>
</template>

<script>
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    import axios from 'axios'
    export default {
        props: {
            dept: { type: String, default: "" },
            deptpublic: { type: String, default: "" },
        },
        data() {
            return {
                loadcount: 0,
                fileList: [],
                visible: false,
                publicdatavisible: false,
                isbg: 'false',
                isfull: false,
                currdiv: 0,
                dd: [],
                i: 0,
                //    maincenterw: 400,
                ishuabu: true,
                style: {
                    width: 1920,//window.screen.width,
                    height: 1080,//window.screen.height,
                    transform: "scaleY(1) scaleX(1) translate(0,0)"
                },
                tablist: [],
            }
        },

        components: {
            EfMain,  //, VCanvas, 
        },
        beforeDestroy() {
            // this.$socket.disconnect();
        },

        created() {
            this.setScale();
            //    this.$socket.connect();

            this.$store.dispatch("tdpublic/setPublicdata", this.$props.deptpublic)

            this.$funAxios.reqPost("PARA_SET", { paras: [{ key: "PARA." + this.$props.deptpublic }] }).then(res => {
                this.tablist = res.data[0].data
                this.loadcount = this.tablist.length;
                res.data[0].data.map((e, i) => {
                    let temp = eval("(" + e.note + ")")
                    if (e.note != null)
                        this.$funAxios.reqPost(temp.api, temp.params).then(res2 => {
                            this.loadcount--
                            e.tabledata = res2.data
                            e.column = [{
                                dataIndex: "no", key: "no", title: "NO", width: 80,
                                scopedSlots: { customRender: "no" }
                            }]
                            for (var key in res2.data[0]) {
                                e.column.push({
                                    dataIndex: key,
                                    key: key,
                                    title: key,
                                    width: 100,
                                })
                            }
                            this.tablist.splice(i, 1, e)
                        })
                })
            })
                .then(() => {
                    if (this.$props.dept) {
                        this.loadcount = 2
                        this.$funAxios.reqPost('APIDATE', {
                            prop: "getdeptfile",
                            id: this.$props.dept
                        }).then(res => {
                            this.loadcount--
                            axios.get(process.env.VUE_APP_API_BASE_URL + '/file/' + res.data[0].note).then(result => {
                                this.loadcount--
                                this.dd = result.data.data.map(e => {
                                    var aaa = () => import('@/components/three3D/' + e.comname + '.vue')
                                    e.com = aaa
                                    this.i = e.key
                                    return e;
                                })
                            })
                        })
                    }
                })
        },
        mounted() {
            let that = this;
            that.setScale();
            window.onresize = () => {
                return (() => {
                    that.isfull = that.checkFull();
                    that.setScale();
                })()
            }
        },
        watch: {
            maincenterh() {
                this.setScale();
            },
            tablist() {
                this.$localForage.setItem('3dpublicdata', this.tablist)

            },
        },
        methods: {
            groupedit() {
                let data = { data: this.dd }
                this.saveJSON(data, "save.json");
            },
            saveJSON(data, filename) {
                if (!data) {
                    alert('data is null');
                    return;
                }
                if (!filename) {
                    filename = 'json.json'
                }
                if (typeof data === 'object') {
                    data = JSON.stringify(data, undefined, 4)
                }
                var blob = new Blob([data], { type: 'text/json' });
                var e = document.createEvent('MouseEvents');
                var a = document.createElement('a');
                a.download = filename;
                a.href = window.URL.createObjectURL(blob);
                a.dataset.downloadurl = ['text/json', a.download, a.href].join(':');
                e.initMouseEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
                a.dispatchEvent(e);
            },
            beforeUpload() {
                return false;
            },
            handle_selectjson(e) {
                this.$funAxios.reqPost('APIDATE', {
                    prop: "deptfile",
                    fileid: e,
                    id: this.$props.dept
                }).then(res => {
                    if (res.data == '1') {
                        this.$message.success("Success")
                    }
                    else {
                        this.$message.error("Error")
                    }
                })
            },

            fullScreen() {
                this.isfull = true;
                this.currdiv = 0;
                var full = document.getElementById('fulls')
                this.launchIntoFullscreen(full)
            },

            //全屏封装
            launchIntoFullscreen(element) {
                if (element.requestFullscreen) {
                    element.requestFullscreen()
                } else if (element.mozRequestFullScreen) {
                    element.mozRequestFullScreen()
                } else if (element.webkitRequestFullscreen) {
                    element.webkitRequestFullscreen()
                } else if (element.msRequestFullscreen) {
                    element.msRequestFullscreen()
                }
            },
            //退出全屏封装
            exitFullscreen() {
                if (document.exitFullscreen) {
                    document.exitFullscreen()
                } else if (document.mozCancelFullScreen) {
                    document.mozCancelFullScreen()
                } else if (document.webkitExitFullscreen) {
                    document.webkitExitFullscreen()
                }
            },
            checkFull() {
                //判断浏览器是否处于全屏状态 （需要考虑兼容问题）
                //火狐浏览器
                var isFull = document.mozFullScreen ||
                    document.fullScreen ||
                    //谷歌浏览器及Webkit内核浏览器
                    document.webkitIsFullScreen ||
                    document.webkitRequestFullScreen ||
                    document.mozRequestFullScreen ||
                    document.msFullscreenEnabled
                if (isFull === undefined) {
                    isFull = false
                }
                return isFull;
            },

            getScale() {
                let w = 0;
                let h = 0;
                if (this.isfull) {
                    w = window.screen.width / 1920;
                    h = window.screen.height / 1080;
                }
                else {
                    h = (this.maincenterh) / 1080;
                    w = this.maincenterw / 1920;
                }
                return { x: w, y: h };
            },
            setScale() {
                let scale = this.getScale();
                this.style.transform = "scaleY(" + scale.y + ") scaleX(" + scale.x + ") translate(0,0)";
            },
        },
        computed: {
            loading() {
                return this.loadcount > 0
            },
            uploadapi() {
                return process.env.VUE_APP_API_BASE_URL + "/file/upload"
            },
            maincenterh() {
                return (this.pageMinHeight - 140)
            },
            maincenterw() {
                return (1920 / 1080) * (this.maincenterh);
            },
            publicdataWidth() {
                return (document.documentElement.clientWidth / 2) * 1;
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),
        },
        directives: {

            styleclick: function (el, binding, vnode) {
                let oDiv = el;
                let that = vnode.context
                let refname = binding.value.refname
                // that.$refs[refname][0].$data.isshow = binding.value.isshow
                let actions = binding.value.actions
                oDiv.onclick = (e) => {
                    if (actions && actions.length > 0) {
                        actions.map(a => {
                            if (a.actiontype == 'click_block') {
                                a.actionobj.map(b => {
                                    that.$refs[b][0].$data.isshow = true
                                })
                            } else if (a.actiontype == 'click_none') {
                                a.actionobj.map(b => {
                                    that.$refs[b][0].$data.isshow = false
                                })
                            }
                            else if (a.actiontype == 'model_click') {
                                a.actionobj.map(b => {
                                    that.$refs[b][0].$data.clickfun = true
                                })
                            }
                            else if (a.actiontype == 'model_unclick') {
                                a.actionobj.map(b => {
                                    that.$refs[b][0].$data.clickfun = false
                                })
                            } else if (a.actiontype == 'click_style') {
                                let temp_style = eval("(" + a.actionstyle + ")")
                                a.actionobj.map(b => {
                                    for (var key in temp_style) {
                                        that.$refs[b][0].$el.style[key] = temp_style[key]
                                    }
                                })
                            } else if (a.actiontype == 'click_return_name') {
                                a.actionobj.map(b => {
                                    that.$refs[b][0].$data.result = that.$refs[refname][0].onMouseClick(e)[0].userData.name;
                                    //    console.log(that.$refs[refname][0].onMouseClick(e)[0].name)
                                    // for (var key in temp_style) {
                                    //     that.$refs[b][0].$el.style[key] = temp_style[key]
                                    // } 
                                })
                            }
                            else if (a.actiontype == 'click_return_binddata') {
                                a.actionobj.map(b => {
                                    let paras = that.$refs[refname][0].onMouseClick(e)[0].userData.binddata;
                                    if (paras != undefined && Object.keys(paras).length > 0) {
                                        var table = that.tablist[Number(paras.key)].tabledata
                                        let result = URL.createObjectURL(table[Number(paras.row) - 1][paras.colname])
                                        // that.$refs[b][0].$data.result = result
                                        that.$refs[b][0].initThree(result);
                                    }
                                    else {
                                        that.$refs[b][0].initThree("");
                                    }

                                })
                            }
                            else if (a.actiontype == 'chose_model') {
                                let model = that.$refs[refname][0].onMouseClick(e)[0].userData.name;
                                if (model != "") {
                                    if (that.$refs[refname][0].$data.modelselected.includes(model)) {
                                        var index = that.$refs[refname][0].$data.modelselected.findIndex(function (item) {
                                            return model == item;
                                        });
                                        that.$refs[refname][0].$data.modelselected.splice(index, 1)
                                    }
                                    else {
                                        that.$refs[refname][0].$data.modelselected.push(model)
                                    }
                                }

                                else {
                                    that.$refs[refname][0].$data.modelselected = []
                                }
                            }
                        })
                    }
                }

                oDiv.onmousedown = () => {
                    if (actions && actions.length > 0) {
                        actions.map(a => {
                            if (a.actiontype == 'click_topage') {
                                that.$refs[refname][0].$data.toroute = a.actionstyle
                                that.$refs[refname][0].click_interval();
                            }
                        })
                    }
                }

            }
        },




    }
</script>

<style lang="less" scoped>
    .success,
    .success:hover,
    .success:visited {
        background-color: @success-color;
        color: white;
    }

    *::-webkit-scrollbar {
        width: 7px;
        height: 7px;
        background-color: transparent;
    }

    /*定义滚动条高宽及背景 高宽分别对应横竖滚动条的尺寸*/
    *::-webkit-scrollbar-track {
        background-color: @primary-3;
    }

    /*定义滚动条轨道 内阴影+圆角*/
    *::-webkit-scrollbar-thumb {
        background-color: @primary-color;
        border-radius: 6px;
    }

    /*定义滑块 内阴影+圆角*/
    .scrollbarHide::-webkit-scrollbar {
        display: none
    }

    .scrollbarShow::-webkit-scrollbar {
        display: block
    }
</style>

<style>
    .maincenter {
        display: flex;
        flex-direction: column;
        justify-content: space-around;
        background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQAQMAAAAlPW0iAAAAA3NCSVQICAjb4U/gAAAABlBMVEXMzMz////TjRV2AAAACXBIWXMAAArrAAAK6wGCiw1aAAAAHHRFWHRTb2Z0d2FyZQBBZG9iZSBGaXJld29ya3MgQ1M26LyyjAAAABFJREFUCJlj+M/AgBVhF/0PAH6/D/HkDxOGAAAAAElFTkSuQmCC");
    }

    #screen {
        z-index: 100;
        transform-origin: 0 0;
        position: relative;
        /* position: fixed;
        left: 50%;
        top: 50%; */
        transition: 0.3s;
        background-color: rgba(6, 13, 20, 0.484);
    }

    #screen:fullscreen {
        background-color: rgb(4, 31, 22);
        transform: scaleY(0.67) scaleX(0.67) translate(0px, 0px) !important
    }

    .combtn:hover {
        cursor: pointer;
    }
</style>
