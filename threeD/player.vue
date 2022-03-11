<template>
    <!-- <ef-main :isMainPage="true" :showtitle="false" style="padding: 0;background-color: aqua;display: flex;justify-content: space-around;flex-direction: column;"> -->

    <div>
        <img v-if="!showvideo" src="./avatar.png" :style="{width:w+'px',height:h+'px'}">
        <div id="myDiv">

            <video v-if="showvideo" ref="videsref" muted id="myVideo" class="video-js vjs-big-play-centered vjs-fluid"
                controls preload="auto" :width="w+'px'">

                <source type="video/mp4" :src="scrs">
            </video>
        </div>
    </div>

    <!-- <div   :style="{width:maxwidth+'px',height:maxheight+'px',backgroundColor:'yellow'}"> -->

    <!-- </div> -->


    <!-- </ef-main> -->
</template>

<script>

    // import EfMain from '@/components/page/main'


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

            }
        },
        components: {
            // EfMain
        },
        created() {

            // let filedata = Array(this.$localForage.getItem('videos'))
            // Promise.all(filedata).then(value => {
            //     if (value[0] != null) {
            //         this.groups = value[0]
            //         this.scrs = URL.createObjectURL(this.groups[0].content)
            //         console.log(this.groups)
            //         this.$nextTick(() => {
            //             console.log(this.groups[0].content)
            //             var vList = this.groups; //视频集
            //             this.curr = 0; //视频下标    如果不需要播放指定视频可以忽略  curr 赋值为0即可
            //             var that = this
            //             this.player = this.$video(document.getElementById('myVideo'), { "playbackRates": [0.25, 0.5, 1, 1.5, 2, 4, 8, 16] }, function onPlayerReady() {
            //                 console.log("1")
            //                 this.play();
            //                 console.log("2")
            //                 this.on('ended', function () {
            //                     if (that.curr >= vList.length) {
            //                         that.curr = 0; //重新循环播放
            //                     }
            //                     that.scrs = URL.createObjectURL(vList[that.curr].content)
            //                     this.src({
            //                         src: that.scrs,
            //                         type: "video/mp4"
            //                     });
            //                     this.load({
            //                         src: that.scrs,
            //                         type: "video/mp4"
            //                     });
            //                     console.log(that.scrs)
            //                     that.curr++; //视频下标加1 
            //                     this.play();
            //                 });
            //             })
            //             this.curr++;
            //         })

            //     }
            // })
            let filedata = Array(this.$localForage.getItem('groups'))
            Promise.all(filedata).then(value => {
                if (value[0] != null) {
                    this.groups = value[0]

                }
            })
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
        methods: {
            handle_show(e) {
                clearInterval(this.timer)
                this.scrs = URL.createObjectURL(e[0].content)

                this.$nextTick(() => {
                    var vList = e; //视频集
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
                            if (that.curr >= vList.length) {
                                // that.curr = 0; //重新循环播放
                                this.dispose();
                                that.showvideo = false;
                                that.hanlde_timer();
                            }
                            else {
                                that.scrs = URL.createObjectURL(vList[that.curr].content)
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
                    console.log(hour + ":" + min)
                    _this.groups.some(e => {
                        //if (e.times == (hour + ":" + min)) {
                              if (e.times == ("13:31")) {
                            // _this.groups = e.val 
                            _this.showvideo = true;
                            _this.handle_show(e.val)
                            return true;
                        }
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
        },
        beforeDestroy() {
            if (this.timer) {
                clearInterval(this.timer);//在vue实例销毁钱，清除我们的定时器
            }
        }
    }
</script>