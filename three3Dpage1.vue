<template>
    <ef-main :isMainPage="true">
        <div class="top" :style="{height: (pageMinHeight-90)+'px'}">
            <!-- class="fullscreen" -->
            <div class="left" style="background-color:rgb(0, 0, 0);width: 10%;overflow-y: scroll;">

                <a-popover placement="right" width="400" trigger="click">
                    <template slot="content">
                        <a-button :class="'success'" @click="handle_showall" size="small">ALL DIV</a-button>
                        <a-button :class="'success'" @click="handle_publicdata" size="small">Public Data</a-button>
                        <a-button :class="'success'" @click="handle_savejson" size="small">保存样式</a-button>
                    </template>
                    <el-button>click 激活</el-button>
                </a-popover>


                <a-button :class="'success'" @click="handle_showall" size="small">ALL DIV</a-button>
                <a-button :class="'success'" @click="handle_publicdata" size="small">Public Data</a-button>
                <a-button :class="'success'" @click="handle_savejson" size="small">保存样式</a-button>
                <!-- <a-button :class="'success'" @click="handle_socket" size="small">Socket</a-button> -->

                <a-upload name="file" :multiple="false" action="#" @change="handle_selectjson" :file-list="fileList"
                    :before-upload="beforeUpload">
                    <a-button :class="'success'" size="small">选取样式</a-button>
                </a-upload>

                <br>
                <a-button type="primary" @click="savestyle" size="small">save</a-button>
                <a-button type="primary" @click="clearstyle" size="small" disabled>Clear</a-button>
                <a-button type="primary" @click="fullScreen()" size="small" :disabled="!ishuabu">全屏</a-button>
                <br>
                <a-button type="primary" @click.once="huabu" size="small" :disabled="publicloading">画布</a-button>
                <br>

                <a-popover placement="right" width="400" trigger="click">
                    <template slot="content">
                        <div v-for="item in settings.btns" :key="item.key">
                            <a-button  type="primary" @click="handle_add(item)"
                            size="small">{{item.btnname}}</a-button><br>
                        </div>
                       
                    </template>
                    <a-button type="link" icon="appstore"></a-button>
                </a-popover>

                <a-button v-for="item in settings.btns" :key="item.key" type="primary" @click="handle_add(item)"
                    size="small">{{item.btnname}}</a-button>
            </div>
            <div class="maincenter" id="maincenter">

                <div v-if="ishuabu" style="width: 100%;height: 200px" id="fulls" ref="fulls"
                    :style="{height:maincenterh+'px',backgroundColor:'rgba(0,0,0,0.4)'}">

                    <div :key="huabui" v-if="ishuabu" id="screen"
                        :style="{'width':style.width+'px','height':style.height+'px','transform': style.transform }">
                        <div v-for="(mainitem,mainindex) in dd " :key="mainitem.key"
                            v-show="mainindex>0?mainindex==curbackground:true">
                            <div @click="sss(item.key)" v-for="(item,index) in mainitem.children" :key="index">
                                <component :ref="item.name+item.key" :vwidth="style.width" :vheight="style.height"
                                    v-drag="{style:item.ss,key:item.key,name:item.name,i:index,maini:mainindex}"
                                    :curdiv="currdiv==item.key && mainindex==curbackground"
                                    v-styleclick="{refname:item.name+item.key,actions:item.ss.actions,isshow:item.ss.isshow}"
                                    :ss="item.ss" :is="item.com">
                                </component>
                            </div>
                        </div>
                    </div>
                </div>
                <div style="display: flex;" v-if="ishuabu">
                    <el-radio-group button-style="solid" v-model="chosedbackground" @change="handle_chosecurbackground">
                        <el-radio-button fill="red" v-for="(item,i) in dd" :label="i" :key="item.key"> 屏幕{{i}}
                        </el-radio-button>
                    </el-radio-group>

                    <!-- <el-button-group>
                        <el-button type="success" @click="handle_chosecurbackground(i)" v-for="(item,i) in dd"
                            :key="item.key">
                            屏幕{{i}}
                        </el-button>
                    </el-button-group> -->
                </div>
            </div>
            <div style="background-color:rgb(92, 92, 92);width: 20%;padding:2px;overflow-y: scroll;">
                <v-css v-if="currdiv>0" @vcss="handle_vcss" :ss="currdivobj.ss" :keyid="currdiv"
                    :actiondivs="treeData[curbackground].children" :name="currdivobj.name"></v-css>
            </div>
        </div>


        <a-drawer :mask="false" :width="dialogwidth" title="DIV LSIT" placement="left" :visible="visible"
            @close="onClose">
            <div :style="{width:'100%',height:drawerheight+'px',overflowY:'scroll'}">
                <a-tree @select="onSelect" :tree-data="treeData" show-icon default-expand-all auto-expand-parent>
                    <template slot="icon" slot-scope="record">
                        <a-icon :type="record.iconname"></a-icon>
                    </template>
                    <template slot="title" slot-scope="record">
                        {{record.name}}--{{record.keyid}}
                        <a-popconfirm title="Are you sure delete ?" ok-text="Yes" cancel-text="No"
                            @confirm="delete_div(record.index)">
                            <a-button type="link" size="small" icon="delete">
                            </a-button>
                        </a-popconfirm>
                    </template>
                </a-tree>
            </div>
        </a-drawer>

        <a-drawer :closable="false" :width="publicdataWidth" title="Public Data" placement="right"
            :visible="publicdatavisible" @close="onClosePublicdata">
            <v-publicdata :tabs="tablist"></v-publicdata>
        </a-drawer>


    </ef-main>
</template>

<script>
    import EfMain from '@/components/page/main'
    import { mapState } from 'vuex'
    import settings from './kanbansetting.json'

    //  import VCanvas from '@/components/three3D/model.vue'
    import VCss from './cssform.vue'
    import VPublicdata from '@/components/three3D/publicdata.vue'
    // import bus from '@/components/three3D/bus.js'

    //  import io from 'socket.io-client'

    export default {
        data() {
            return {
                imgsrc: process.env.VUE_APP_API_WEB_URL + '/storage/eee.jpeg',
                imgsrc1: process.env.VUE_APP_API_WEB_URL + '/modelapi/eee.jpeg',
                fileList: [],
                huabui: 0,
                visible: false,
                publicdatavisible: false,
                isbg: 'false',
                isfull: false,
                currdiv: 0,
                currdivobj: {},
                as: {
                    left: 0,
                    width: 100,
                    height: 100,
                    url: "",
                    backgroundColor: "",
                    zindex: 500,
                    content: "",
                    fontSize: 20,
                    actions: [],
                    modelstyle: [],
                },
                dd: [],
                curbackground: 1,
                i: 0,
                maincenterh: 400,
                ishuabu: false,
                style: {
                    width: 1920,//window.screen.width,
                    height: 1080,//window.screen.height,
                    transform: "scaleY(1) scaleX(1) translate(0,0)"
                },
                settings,
                tablist: [],
                loadcount: -1,

                chosedbackground: 1,
            }
        },

        components: {
            EfMain, VCss,//, VCanvas,
            VPublicdata
        },
        beforeDestroy() {
            //  this.$socket.disconnect();
        },
        created() {
            //   this.$socket.connect();
            let filedata = Array(this.$localForage.getItem('3dstyle'))
            Promise.all(filedata).then(value => {
                if (value[0] != null) {
                    if (value[0].length > 0) {
                        this.dd = value[0].map(e => {
                            var aaa = () => import('@/components/three3D/' + e.comname + '.vue')
                            e.com = aaa
                            this.i = this.i > e.key ? this.i : e.key
                            e.children.map(m => {
                                this.i = this.i > m.key ? this.i : m.key
                                var temp = () => import('@/components/three3D/' + m.comname + '.vue')
                                m.com = temp
                                return m
                            })
                            return e;
                        })
                    }
                }
            })

            this.$store.dispatch("tdpublic/setPublicdata", 528)

            this.$funAxios.reqPost("PARA_SET", { paras: [{ key: "PARA.528" }] }).then(res => {
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
                                    scopedSlots: { customRender: key }
                                })
                            }
                            this.tablist.splice(i, 1, e)
                        })
                })
            })
        },
        watch: {
            tablist() {
                this.$localForage.setItem('3dpublicdata', this.tablist)
            },

        },
        mounted() {
            let that = this;
            that.setScale();
            /*窗口改变事件*/
            window.onresize = () => {
                return (() => {
                    that.isfull = that.checkFull();
                    that.setScale();
                })()
            }
        },
        methods: {
            beforeUpload() {
                return false;
            },
            handle_selectjson(info) {
                let fileList = [...info.fileList];
                fileList = fileList.slice(-1);
                this.fileList = fileList;
                const reader = new FileReader();
                reader.readAsText(info.file);
                reader.onload = () => {
                    this.dd = JSON.parse(reader.result).data.map(e => {
                        var aaa = () => import('@/components/three3D/' + e.comname + '.vue')
                        e.com = aaa
                        this.i = this.i > e.key ? this.i : e.key
                        e.children.map(m => {
                            this.i = this.i > m.key ? this.i : m.key
                            var temp = () => import('@/components/three3D/' + m.comname + '.vue')
                            m.com = temp
                            return m
                        })
                        return e;
                    })
                };
            },
            handle_savejson() {
                let data = { data: this.dd }
                this.saveJSON(data, "3Dkanban_" + new Date().format("Y-m-d H:i:s") + ".json");
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

            handle_publicdata() {
                this.publicdatavisible = true
            },
            delete_div(e) {
                this.currdiv = 0;
                let arr = e.split('-')
                if (arr.length == 2) {
                    this.dd.splice(arr[1], 1)
                }
                else if (arr.length == 3) {
                    this.dd[this.curbackground].children.splice(arr[2], 1)
                }

            },
            handle_chosecurbackground() {
                this.curbackground = this.chosedbackground;
                this.currdiv = this.dd[this.chosedbackground].key
                this.currdivobj = this.dd[this.chosedbackground]
            },
            onSelect(selectedKeys, info) {
                if (info.selected) {
                    let arr = selectedKeys[0].split('-')
                    this.curbackground = arr[1]
                    if (arr.length == 2) {
                        this.currdiv = this.dd[arr[1]].key
                        this.currdivobj = this.dd[arr[1]]
                    }
                    else if (arr.length == 3) {
                        this.currdiv = this.dd[arr[1]].children[arr[2]].key
                        this.currdivobj = this.dd[arr[1]].children[arr[2]]
                    }
                }
            },
            onClose() {
                this.visible = false;
            },
            onClosePublicdata() {
                this.publicdatavisible = false;
            },
            handle_showall() {
                this.visible = true;
            },
            clearstyle() {
                this.$localForage.setItem("3dstyle", [])
            },
            savestyle() {
                var temp = [];
                this.dd.map(e => {
                    temp.push({
                        comname: e.comname,
                        ss: e.ss, key: e.key, name: e.name,
                        iconname: e.iconname,
                        children: e.children.map(m => {
                            return {
                                comname: m.comname,
                                ss: m.ss, key: m.key, name: m.name,
                                iconname: m.iconname,
                            }
                        })
                    })
                })
                this.$localForage.setItem("3dstyle", temp)
            },
            huabu() {
                this.ishuabu = true
                this.maincenterh = (1080 / 1920) * (document.getElementById('maincenter').clientWidth)
                this.setScale();
                if (this.dd.length == 0) {
                    var aaa = () => import('@/components/three3D/background.vue')
                    this.i++;
                    var keys = this.i
                    this.dd.push({
                        comname: "background", iconname: "appstore",
                        com: aaa, ss: { ...this.as },
                        key: keys, name: "background", children: [],
                    })

                    this.curbackground = 0




                }
            },

            sss(e) {
                if (!this.checkFull()) {
                    //this.curbackground = maini
                    this.currdiv = e
                    var i = 0;
                    this.dd[this.curbackground].children.some((a, index) => {
                        if (a.key == e) {
                            i = index
                            return true
                        }
                    })
                    this.currdivobj = this.dd[this.curbackground].children[i];
                }
            },
            handle_vcss(e) {
                this.dd[this.curbackground].children.some(a => {
                    if (a.key == e.key) {
                        a.ss = e.style
                        return true
                    }
                })
            },
            handle_add(e) {
                var aaa = () => import('@/components/three3D/' + e.comname + '.vue')
                this.i++;
                var keys = this.i
                if (e.key == "background") {
                    this.dd.push({
                        comname: e.comname, iconname:
                            e.iconname, com: aaa, ss: { ...this.as, ...e.expandpara },
                        key: keys, name: e.name, children: []
                    })
                    this.curbackground = this.dd.length - 1
                }
                else {
                    var obj = { comname: e.comname, iconname: e.iconname, com: aaa, ss: { ...this.as, ...e.expandpara }, key: keys, name: e.name }
                    this.dd[this.curbackground].children.push(obj)
                    this.currdiv = keys
                    this.currdivobj = obj
                }
            },

            fullScreen() {
                this.isfull = true;
                this.currdiv = 0;
                this.currdivobj = {};
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
                    w = document.getElementById('maincenter').clientWidth / 1920;
                    h = this.maincenterh / 1080;
                }
                return { x: w, y: h };
            },
            setScale() {
                let scale = this.getScale();
                this.style.transform = "scaleY(" + scale.y + ") scaleX(" + scale.x + ") translate(0,0)";
            },
        },
        computed: {

            publicloading() {
                return this.loadcount > 0
            },
            loading() {
                return this.dd.length > 0
            },
            treeData() {
                var res = [];
                this.dd.map((e, i) => {
                    if (e.name == "background") {
                        res.push({
                            index: 0 + '-' + i,
                            name: e.name,
                            key: 0 + '-' + i,
                            keyid: e.key,
                            iconname: e.iconname,
                            scopedSlots: {
                                icon: 'icon',
                                title: 'title'
                            },
                            children: e.children.map((m, index) => {
                                return {
                                    // title: e.name,
                                    index: 0 + '-' + i + '-' + index,
                                    name: m.name,
                                    key: 0 + '-' + i + '-' + index,
                                    keyid: m.key,
                                    iconname: m.iconname == "" ? "ellipsis" : m.iconname,
                                    scopedSlots: {
                                        icon: 'icon',
                                        title: 'title'
                                    }
                                }
                            }),
                        })
                    }

                })
                return res
            },
            drawerheight() {
                return this.pageMinHeight - 30
            },
            maxheight() {
                return window.screen.height
            },
            maxwidth() {
                return window.screen.width
            },
            dialogwidth() {
                return (document.documentElement.clientWidth / 5) * 1;
            },
            publicdataWidth() {
                return (document.documentElement.clientWidth / 2) * 1;
            },
            modalHeight() {
                return (document.documentElement.clientHeight) - 200;
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),
            scroll() {
                return { x: 1100, y: (this.pageMinHeight - 200) / 2 }
            },
        },
        directives: {
            drag: {
                // 指令的定义
                bind: function (el, binding, vnode) {
                    let that = vnode.context
                    let oDiv = el;  // 获取当前元素   
                    var cw = document.getElementById('maincenter').clientWidth
                    var temp1 = (1080 / 1920) * (cw);

                    let sx = cw / 1920
                    let sy = temp1 / 1080;
                    if (binding.value.name != 'bg') {
                        oDiv.onmousedown = (e) => {
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

                            if (!isFull && that.curbackground == binding.value.maini) {
                                binding.value.currdiv = binding.value.key
                                // 算出鼠标相对元素的位置
                                let disX = e.clientX / sx - oDiv.offsetLeft;
                                let disY = e.clientY / sy - oDiv.offsetTop;
                                if (oDiv.offsetLeft <= 0) {
                                    oDiv.style.left = 0;
                                }
                                document.onmousemove = (e) => {
                                    // 用鼠标的位置减去鼠标相对元素的位置，得到元素的位置
                                    if (oDiv.offsetLeft >= 0) {
                                        let left = e.clientX / sx - disX;
                                        let top = e.clientY / sy - disY;
                                        // oDiv.style.left = left + 'px';
                                        // oDiv.style.top = top + 'px';
                                        that.dd[that.curbackground].children[binding.value.i].ss.left = left
                                        that.dd[that.curbackground].children[binding.value.i].ss.top = top
                                    }
                                    else {
                                        document.onmousemove = null;
                                        document.onmouseup = null;
                                    }

                                };
                                document.onmouseup = () => {
                                    document.onmousemove = null;
                                    document.onmouseup = null;
                                };

                            }

                        }
                    }
                }
            },
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
                                })
                            }
                            else if (a.actiontype == 'click_return_binddata') {
                                a.actionobj.map(b => {
                                    let paras = that.$refs[refname][0].onMouseClick(e)[0].userData.binddata;
                                    if (paras != undefined && Object.keys(paras).length > 0) {
                                        var table = that.tablist[Number(paras.key)].tabledata
                                        let result = table[Number(paras.row) - 1][paras.colname]
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
                            else if (a.actiontype == 'click_changebg') {
                                if (that.checkFull()) {
                                    let temp_style = eval("(" + a.actionstyle + ")")
                                    that.chosedbackground = temp_style.index
                                    that.handle_chosecurbackground()
                                }
                            }
                        })
                    }
                    //    that.$refs[refname][0].$data.result = "123"
                    // that.$refs[refname][0].$el.style.display="none"
                }

                // oDiv.onmousedown = () => {
                //     if (actions && actions.length > 0) {
                //         actions.map(a => {
                //             if (a.actiontype == 'click_topage') {
                //                 that.$refs[refname][0].$data.toroute = a.actionstyle
                //                 that.$refs[refname][0].click_interval();
                //             }
                //         })
                //     }
                // }

            },

        }
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
    .top {
        display: flex;
        background-color: rgb(97, 97, 97);
    }

    .left {
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
    }

    .left button {
        margin: 5px;
    }

    .maincenter {
        display: flex;
        flex-direction: column;
        justify-content: center;
        width: 70%;
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
        background-color: rgba(127, 255, 212, 0.057);
    }

    #screen:fullscreen {
        background-color: rgb(4, 31, 22);
        transform: scaleY(0.67) scaleX(0.67) translate(0px, 0px) !important
    }

    .combtn:hover {
        cursor: pointer;
    }

    .test {
        color: #00ff77;
    }
</style>
