<template>
    <!-- <ef-main :isMainPage="true" :showtitle="false" style="padding: 0;background-color: aqua;display: flex;justify-content: space-around;flex-direction: column;"> -->

    <div>

        <img v-if="!showvideo" :src="icons+'/file/'+screenpic+'/large' " :style="{width:w+'px',height:h+'px'}">

        <!-- <img src="./avatar.png" :style="{width:w+'px',height:h+'px'}"> -->
        <div id="myDiv">
            <video v-if="showvideo" ref="videsref" muted id="myVideo" class="video-js vjs-big-play-centered vjs-fluid"
                controls preload="auto" :width="w+'px'">
                <source type="video/mp4" :src="scrs">
            </video>
        </div>
    </div>
</template>

<script>

    // import EfMain from '@/components/page/main'

    import { reqPost } from '@/services/base'
    const DataSet = require('@antv/data-set')

    export default {
        props: "",
        data() {
            return {
                groups: [],
                videoi: 0,


                scrs: "",

                curr: 0,
                isvideo: true,
                player: null,

                loading: false,
                source: "",

                showvideo: false,
                w: document.documentElement.clientWidth,
                h: document.documentElement.clientHeight,

                screenpic: "",


            }
        },
        components: {
            // EfMain
        },
        created() {
            if (Object.values(this.$route.query).length > 0) {
                this.screenpic = this.$route.query.screenpic;
                this.channel = this.$route.query.channel;
                this.handle_getplay();
            }
            else {
                //
            }
        },
        mounted() {
            this.hanlde_timer();
            const that = this
            window.onresize = () => {
                return (() => {
                    that.w = document.documentElement.clientWidth
                    that.h = document.documentElement.clientHeight
                })()
            }
        },
        watch: {
            day() {
                this.handle_getplay();
            },
        },
        methods: {
            handle_getplay() {
                reqPost("VBOARD", {
                    prop: "getplayer",
                    playdate: this.year + "-" + this.month + "-" + this.day,
                    channel: this.channel
                }).then(res => {
                    this.loadcount--;
                    res.data.map(e => {
                        e.seq = Number(e.seq)
                        return e
                    })

                    const dv = new DataSet.DataView().source(res.data).transform({
                        type: 'sort-by',
                        fields: ['timefrom'], // 根据指定的字段集进行排序，与lodash的sortBy行为一致

                    })
                        .transform({
                            type: 'partition',
                            groupBy: ['playbillid'],
                            orderBy: ['seq']
                        })
                    var obj = dv.rows;
                    var temp = [];
                    for (let key in obj) {
                        temp.push({
                            playbillid: obj[key][0].playbillid,
                            timefrom: obj[key][0].timefrom,
                            timeto: obj[key][0].timeto,
                            val: []
                        })
                    }
                    temp.map(m => {
                        res.data.map(n => {
                            if (m.playbillid == n.playbillid) {
                                m.val.push(n.playerid)
                            }
                        })
                        return m
                    })
                    this.$localForage.setItem('groupsa', temp)
                    this.groups = temp

                })
            },

            handle_show(e) {
                clearInterval(this.timer)
                this.scrs = this.icons + '/file/' + e[0] // URL.createObjectURL(e[0].content) 
                this.$nextTick(() => {
                    this.curr = 0; //视频下标    如果不需要播放指定视频可以忽略  curr 赋值为0即可

                    var that = this
                    //document.getElementById('myVideo')   
                    if (document.getElementById('myDiv').childNodes.length == 0) {
                        this.scrs = URL.createObjectURL(e[0].content)
                        document.getElementById('myDiv').appendChild(this.$refs.videsref)
                    }
                    this.player = this.$video(this.$refs.videsref, { "playbackRates": [0.25, 0.5, 1, 1.5, 2, 4, 8, 16], autoplay: true, }, function onPlayerReady() {
                        this.play();
                        this.on('ended', function () {
                            if (that.curr >= e.length) {
                                // that.curr = 0; //重新循环播放
                                this.dispose();
                                that.showvideo = false;
                                that.hanlde_timer();
                            }
                            else {
                                that.scrs = that.icons + '/file/' + e[that.curr]
                                this.src({
                                    src: that.scrs,
                                    type: "video/mp4"
                                });
                                this.load({
                                    src: that.scrs,
                                    type: "video/mp4"
                                });
                                that.curr++; //视频下标加1 
                                this.play();
                            }

                        });
                    })
                    this.curr++;
                })
            },

            hanlde_timer() {
                var _this = this;
                this.timer = setInterval(function () {
                    //_this.handle_show()
                    _this.date = new Date();//修改数据date 
                    var hour = new Date().getHours() < 10 ? ("0" + new Date().getHours()) : new Date().getHours();
                    var min = new Date().getMinutes() < 10 ? ("0" + new Date().getMinutes()) : new Date().getMinutes();
                    var current = Number(hour) * 60 + Number(min)
                    _this.groups.some(e => {
                        //if (e.times == (hour + ":" + min)) {
                        var fm = Number(e.timefrom.split(":")[0]) * 60 + Number(e.timefrom.split(":")[1])
                        var to = Number(e.timeto.split(":")[0]) * 60 + Number(e.timeto.split(":")[1])
                        if (fm < current && to > current) {
                            //
                            _this.showvideo = true;
                            _this.handle_show(e.val)
                            return true;
                        }
                        else {
                            _this.showvideo = false;
                        }
                        // if (e.times == ("13:31")) {
                        //     // _this.groups = e.val 

                        // }
                    })
                }, 10000);
            },
        },
        computed: {
            maxwidth() {
                return (document.documentElement.clientWidth);
            },
            maxheight() {
                return (document.documentElement.clientHeight);
            },
            plant() {
                return this.$route.fullPath.split('/')[3]
            },
            icons() {
                return process.env.VUE_APP_API_WEB_URL;
            },
            year() {
                return new Date().getFullYear()

            },
            month() {
                return new Date().getMonth() + 1
            },
            day() {
                return new Date().getDate()
            },
        },
        beforeDestroy() {
            if (this.timer) {
                clearInterval(this.timer);//在vue实例销毁钱，清除我们的定时器
            }
        }
    }
</script>