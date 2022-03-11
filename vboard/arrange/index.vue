<template>
    <ef-main :isMainPage="true" :loading="loading" :form-items="searchform" @submit="handle_search" @adds="handle_edit"
        :adds="true">
        <div id="aa" :style="{height: calendarheight, overflowY:'auto'}">
            <a-calendar v-model="calendar">
                <!-- :header-render="headerRender" -->
                <template slot="dateCellRender" class="events" slot-scope="value">
                    <div class="content" style="height: 100%;width: 100%;overflow-x:auto"
                        @click="handle_saveadd('date',value)">

                        <a-tag color="green" v-for="(arr,i) in datearrange(value)" :key="i+'date'">
                            <span @click="handle_toplaylist(arr)"> {{arr.playbillname}}</span>
                            <a-button @click="handle_delete(arr)" type="link" size="small">
                                <a-icon type="delete"></a-icon>
                            </a-button>
                        </a-tag>
                    </div>
                </template>

                <template slot="monthCellRender" class="events" slot-scope="value">
                    <div style="height: 100%;width: 100%;overflow-y: auto;" @click="handle_saveadd('month',value)">
                        <a-tag color="green" v-for="(arr,i) in montharrange(value)" :key="i+'month'">
                            {{arr.playdate}} {{arr.playbillname}}
                        </a-tag>
                    </div>
                    <div style="position:absolute;right: 15px;bottom:0">

                        <a-popconfirm title="Are you sure delete All Month?" ok-text="Yes" cancel-text="No"
                            @confirm="handle_deletemonth(value)">
                            <a-button size="small" type="link">
                                <a-icon type="delete"></a-icon> 清空/月
                            </a-button>
                        </a-popconfirm>
                    </div>
                </template>
            </a-calendar>
        </div>
        <div id="rianley" v-show="isadd">
            <a-tag color="green">{{cur}}</a-tag>
        </div>

        <a-drawer width="640" placement="right" :closable="false" :visible="visible" @close="onClose">

            <p :style="[pStyle, pStyle2]">
                Edit
            </p>
            <div v-for="(item,index) in playbilllist" :key="index">
                <p :style="pStyle">
                    {{item.timefrom}}-{{item.timeto}}
                    {{item.name}}
                </p>
                <div style="width: 100%;
                    display: flex;justify-content: space-between;  flex-wrap: wrap;">
                    <div style="margin:5px;" v-for="(con,i) in item.content" :key="i">
                        <a-tag color="green">{{con.seq}}-{{con.name}}</a-tag>
                    </div>
                </div>

                <div>
                    <a-button size="small" type="primary" @click="handle_add(item)">添加到指定</a-button>
                </div>
                <a-divider />
            </div>
        </a-drawer>
    </ef-main>

</template>

<script>
    import EfMain from '@/components/page/main'
    import { reqPost } from '@/services/base'
    // import { moment } from '@/utils/util'
    import { mapState } from 'vuex'
    export default {
        props: {
            tabledate: { type: Array, default: () => [] },
        },
        data() {
            return {
                calendar: null,
                values: {},
                loadcount: 0,

                channels: [],

                searchform: [{
                    key: 'channel',
                    type: 'select',
                    options: [],
                    value: ""
                }],

                playbilllist: [],
                cur: "",

                visible: false,
                pStyle: {
                    fontSize: '16px',
                    color: 'rgba(0,0,0,0.85)',
                    lineHeight: '24px',
                    display: 'block',
                    marginBottom: '16px',
                },
                pStyle2: {
                    marginBottom: '24px',
                },
                playbill: "",

                arranges: [],
                isadd: false,
                actionstatu: "", //add,

            }
        },
        created() {


            this.loadcount = 1;
            reqPost("VBOARD", { prop: "getchannel" }).then(res => {
                this.loadcount--;
                this.channels = res.data.map(e => {
                    return {
                        key: e.id,
                        title: e.name,
                    }
                });
                this.searchform[0].options = this.channels;
                this.searchform[0].value = this.channels[0].key
                this.handle_search({ channel: this.searchform[0].value });
                this.get_arrange({ channel: this.searchform[0].value })

            })

        },
        computed: {
            plant() {
                return this.$route.fullPath.split('/')[3]
            },
            ...mapState('setting', ['lang', 'pageMinHeight']),
            loading() {
                return this.loadcount > 0;
            },
            calendarheight() {
                return (this.pageMinHeight - 95) + 'px'
            },
        },
        components: {
            EfMain
        },
        beforeDestroy() {
            document.getElementById("rianley").remove();
        },
        methods: {
            handle_toplaylist(item) {
                var temp = this.$route.fullPath.split('/')
                this.$router.push("/" + temp[1] + "/" + temp[2] + "/" + this.plant
                    + "/playcontent?playbill=" + item.playbillid
                    + "&channel=" + item.channelid
                    + "&time=" + item.timefrom + "-" + item.timeto
                    + "&name=" + item.playbillname
                    + "&channelname=" + item.channelname)
            },

            handle_deletemonth(e) {
                reqPost("VBOARD", { prop: "deletearrangemonth", month: e.format("yyyy-MM") }).then(res => {
                    if (res.data > 0) {
                        this.$message.success("Clear Success")
                        this.get_arrange({ channel: this.searchform[0].value });
                    } else {
                        this.$message.error("Error")
                    }
                })

            },
            handle_delete(e) {
                reqPost("VBOARD", { prop: "deletearrange", id: e.arrangeid }).then(res => {
                    if (res.data > 0) {
                        this.$message.success(e.playdate + " " + e.playbillname + "Delete Success")

                        this.arranges.splice(e.i, 1)
                    } else {
                        this.$message.error("Error")
                    }
                })
            },
            datearrange(value) {
                var temp = [];
                this.arranges.map((e, index) => {
                    if (e.playdate == value.format("yyyy-MM-DD")) {
                        e.i = index;
                        temp.push(e)
                    }
                })
                return temp
            },

            montharrange(value) {
                var temp = [];
                this.arranges.map(e => {
                    if (e.playdate.slice(0, 7) == value.format("yyyy-MM")) {
                        temp.push(e)
                    }
                })
                return temp
            },

            get_arrange(values) {
                reqPost("VBOARD", { prop: " getarrange", channel: values.channel })
                    .then(res => {
                        this.arranges = res.data;
                    })
            },

            handle_saveadd(type, e) {
                switch (this.actionstatu) {
                    case "add":
                        if (type == "date") {
                            this.handle_date(e.format("yyyy-MM-DD"));
                        }
                        else if (type == "month") {
                            this.handle_month(e.format("yyyy-MM-DD"));
                        }
                        else {
                            //
                        }
                        break;
                    default:
                        break;
                }


            },
            handle_month(e) {
                var params = {
                    prop: "insertarrange",
                    type: "month",
                    year: e.split('-')[0],
                    month: e.split('-')[1],
                    playbill: this.playbill,
                    playdate: e
                }
                this.handle_request(params);
            },

            handle_date(e) {
                var params = {
                    prop: "insertarrange",
                    type: "date",
                    playbill: this.playbill,
                    playdate: e
                }
                this.handle_request(params);
            },

            handle_request(params) {
                reqPost("VBOARD", params).then(res => {
                    if (res.data == true) {
                        this.$message.success(params.playdate + " " + this.cur + " Add Success")
                        this.get_arrange({ channel: this.searchform[0].value });
                    }
                    else if (res.data == 'exist') {
                        this.$message.warning("已存在")
                    } else {
                        this.$message.error("Error")
                    }
                })
            },
            handle_add(e) {
                this.visible = false;
                this.isadd = true;
                this.actionstatu = "add"
                this.cur = e.name
                this.playbill = e.id
            },

            onClose() {
                this.visible = false;
            },

            handle_edit() {
                this.visible = true;
            },

            handle_search(values) {
                this.loadcount = 2;
                reqPost("VBOARD", { prop: "getplaybill", channel: values.channel }).then(res => {
                    this.loadcount--;
                    this.playbilllist = res.data.map(m => {
                        m.content = [];
                        return m;
                    });
                    reqPost("VBOARD", { prop: "getplaylistall", channel: values.channel }).then(res => {
                        this.loadcount--;

                        res.data.map(e => {
                            this.playbilllist.some(o => {
                                if (e.playbill == o.id) {
                                    o.content.push(e)
                                    return true;
                                }
                            })
                            return e;
                        })
                        this.get_arrange(values);
                    })
                })
            },

        },
        mounted() {
            document.getElementById('aa').onmousemove = function (e) {
                var app = document.body;
                var rianley = document.getElementById("rianley");
                app.appendChild(rianley)
                rianley.style.left = e.clientX + 10 + "px";
                rianley.style.top = e.clientY + 10 + "px";
                rianley.style.cursor = "pointer";
            }

            document.getElementById('aa').oncontextmenu = ((e) => {
                if (e && e.preventDefault) {
                    //阻止默认浏览器动作(W3C)
                    e.preventDefault()
                    this.isadd = false
                    this.cur = "";
                    this.playbill = "";
                    this.actionstatu = "";
                } else {
                    //IE中阻止函数器默认动作的方式
                    window.event.returnValue = false
                }
                return false
            })
        },
    };
</script>
<style scoped lang="less">
    .events {
        list-style: none;
        margin: 0;
        padding: 0;
    }

    .events .ant-badge-status {
        overflow: hidden;
        white-space: nowrap;
        width: 100%;
        text-overflow: ellipsis;
        font-size: 12px;
    }

    .notes-month {
        text-align: center;
        font-size: 28px;
    }

    .notes-month section {
        font-size: 28px;
    }

    #rianley {

        position: absolute;
        left: 0;
        top: 0;
    }

    *::-webkit-scrollbar {
        width: 7px;
        height: 10px;
        background-color: transparent;
    }

    /*定义滚动条高宽及背景 高宽分别对应横竖滚动条的尺寸*/
    *::-webkit-scrollbar-track {
        background-color: @primary-4;
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



    .content::-webkit-scrollbar {
        width: 2px;
        height: 5px;
        background-color: transparent;
    }

    /*定义滚动条高宽及背景 高宽分别对应横竖滚动条的尺寸*/
    .content::-webkit-scrollbar-track {
        background-color: @primary-1;
    }

    /*定义滚动条轨道 内阴影+圆角*/
    .content::-webkit-scrollbar-thumb {
        background-color: @primary-4;
        border-radius: 6px;
    }
</style>