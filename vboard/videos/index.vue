<template>
    <ef-main :loading="loading" :isMainPage="true" @adds="show_weather" :adds="true" :form-items="searchform"
        @submit="handle_search" :key="formi">

        <v-list @playervideo="playervideo" @action="handle_action" :videos="videos"></v-list>

        <a-drawer :width="dialogwidth" placement="right" :closable="false" :visible="visible" @close="onClose">
            <v-form :loading="loading" ref="formchild" @filechange="filechange" title="上传视频" :form-items="formItems"
                @submit="submit" :rules="rules" :form="form">
            </v-form>
        </a-drawer>

        <a-modal :visible="videoshow" :confirm-loading="confirmLoading" @cancel="handleCancel" @ok="handleCancel"
            :footer="null">
            <video-player v-if="videoshow" :key="videoi" class="video-player vjs-custom-skin" ref="videoPlayer"
                :playsinline="true" :options="playerOptions" @play="onPlayerPlay($event)" @pause="onPlayerPause($event)"
                @ended="playerend" @ready="playerReadied">
            </video-player>
        </a-modal>


    </ef-main>

</template>

<script>
    import { videoPlayer } from 'vue-video-player'
    import EfMain from '@/components/page/main'
    import VList from './list.vue'
    import VForm from '@/components/page/drawform'
    import { merge, moment } from '@/utils/util'
    import { reqPost } from '@/services/base'

    const i18n = require('./i18n')
    const rules = {
        name: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
        content: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
        channel: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
    }

    const form = {
        name: "",
        desc: "",
        cover: "",
        content: [],
        channel: "",
    }

    export default {
        i18n: merge(require('@/i18n'), i18n),
        data() {
            return {
                moment,
                groupvisible: false,
                videoi: 0,
                playerOptions: {
                    playbackRates: [0.7, 1.0, 1.5, 2.0], //播放速度
                    autoplay: true, //如果 true，浏览器准备好时开始回放
                    muted: false, //默认情况下将会消除任何音频
                    loop: false, //导致视频一结束就重新开始
                    preload: 'auto', //建议浏览器在 <video> 加载元素后是否应该开始下载视频数据，auto 浏览器选择最佳行为，立即开始加载视频（如果浏览器支持）
                    language: 'zh-CN',
                    aspectRatio: '16:9', //将播放器置于流畅模式，并在计算播放器的动态大小时使用该值，值应该代表一个比例 - 用冒号分隔的两个数字（例如 "16:9" 或 "4:3"）
                    fluid: true, //当 true 时，Video.js player 将拥有流体大小。换句话说，它将按比例缩放以适应其容器
                    sources: [],
                    poster: "/avatar.png", //你的封面地址
                    width: document.documentElement.clientWidth,
                    notSupportedMessage: '此视频暂无法播放，请稍后再试', //允许覆盖 video.js 无法播放媒体源时显示的默认信息
                    controlBar: {
                        timeDivider: true,
                        durationDisplay: true,
                        remainingTimeDisplay: false,
                        fullscreenToggle: true //全屏按钮
                    }
                },
                flag: false,
                visible: false,
                rules, form,
                videos: [],
                actionstatus: "",
                videoshow: false,

                confirmLoading: false,

                // mockData: [],
                targetKeys: [],
                hour: "12",
                min: "00",

                groups: [],
                groupname: "",

                date: new Date(),

                isFullscreen: false,
                groupi: 0,
                loadcount: 0,
                formi: 0,
                channels: [],
                axiosprop: { "add": "insertcontent", "update": "updatecontent" },
                icons: process.env.VUE_APP_API_WEB_URL,
            }
        },

        components: {
            videoPlayer, EfMain, VList, VForm
        },
        created() {
            this.handle_get();
            this.handle_searchgroup();
            reqPost("VBOARD", { prop: "getchannel" }).then(res => {
                this.loadcount--;
                this.channels = res.data.map(e => {
                    return {
                        key: e.id,
                        title: e.name,
                    }
                });
                this.formi++;
            })
        },

        mounted() {

        },
        methods: {
            // onPlayerPlay(player) {
            //     console.log(player)
            // }, 
            handle_search(values) {
                this.loadcount = 1;
                reqPost("VBOARD", { prop: "getcontent", channel: values.channel }).then(res => {
                    this.loadcount--;
                    this.videos = res.data;
                })
            },
            handle_get() {
                this.loadcount = 1;
                reqPost("VBOARD", { prop: "getcontent" }).then(res => {
                    this.loadcount--;
                    this.videos = res.data;
                })
            },

            handle_searchgroup() {
                let filedata = Array(this.$localForage.getItem('groups'))
                Promise.all(filedata).then(value => {
                    if (value[0] != null) {
                        this.groups = value[0]
                    }
                })
            },
            groupClose() {
                this.groupvisible = false;
            },
            onClose() {
                this.visible = false;
                this.$refs.formchild.clearForm();
            },

            onPlayerPause(player) {
                console.log(player)
            },

            show_weather() {
                this.actionstatus = "add"
                this.visible = true;
            },

            playerReadied(e) {
                console.log(e)
            },
            onPlayerPlay() {
                //
            },
            playerend() {
                // 
            },
            filechange(e) {
                this.form.name = e[0].name
                var blob = new Blob([e[0].originFileObj], { type: e[0].type })
                let dataURL = '';
                let video = document.createElement("video");
                video.setAttribute('crossOrigin', 'anonymous');//处理跨域
                video.setAttribute('src', URL.createObjectURL(blob));
                video.setAttribute('width', 1920);
                video.setAttribute('height', 1080);
                video.currentTime = 1;
                video.addEventListener('loadeddata', () => {
                    let canvas = document.createElement("canvas")
                    let width = video.width  //canvas的尺寸和图片一样
                    let height = video.height
                    canvas.width = width;
                    canvas.height = height;
                    canvas.getContext("2d").drawImage(video, 0, 0, width, height); //绘制canvas
                    dataURL = canvas.toDataURL('image/jpeg'); //转换为base64  
                    var file = this.dataURLtoFile(dataURL, "cover.jpg");
                    let formData = new FormData();
                    formData.append('file', file);
                    this.loadcount = 1;
                    reqPost('FILE_UPLOAD', formData).then((res) => {
                        this.loadcount--;
                        if (res.code == 200) {
                            this.form.cover = res.id;
                        }
                        else {
                            this.$message.error("上傳失敗")
                        }
                    })
                });
            },
            //将base64转换为文件
            dataURLtoFile(dataurl, filename) {
                var arr = dataurl.split(','),
                    mime = arr[0].match(/:(.*?);/)[1],
                    bstr = atob(arr[1]),
                    n = bstr.length,
                    u8arr = new Uint8Array(n);
                while (n--) {
                    u8arr[n] = bstr.charCodeAt(n);
                }
                return new File([u8arr], filename, { type: mime });
            },

            submit(value) {
                if (value.content[0].originFileObj != undefined) {
                    let formData = new FormData();
                    formData.append('file', value.content[0].originFileObj);
                    this.loadcount = 1;
                    reqPost('FILE_UPLOAD', formData).then((res) => {
                        this.loadcount--;
                        if (res.code == 200) {
                            this.handle_ok(value, res.id);
                        }
                        else {
                            this.$message.error("上傳失敗")
                        }
                    })
                }
                else {
                    this.handle_ok(value, this.form.fileid);
                }
            },

            handle_ok(value, fileid) {
                var paras = {
                    ...value,
                    ...{ fileid: fileid },
                    ...{ prop: this.axiosprop[this.actionstatus] },
                    ...{ id: form.id }
                }
                reqPost("VBOARD", paras).then(res => {
                    if (res.data > 0) {
                        var that = this
                        this.$message.success("Success")
                        this.$refs.formchild.clearForm();
                        this.handle_get();
                        this.actionstatus == "add" ?
                            (function () {
                                // that.$nextTick(() => {
                                //     form.seq = that.tabledata.length + 1;
                                // })
                            }()) :
                            (this.actionstatus == "update" ?
                                (function () { that.visible = false; }()) :
                                "")
                    }
                    else {
                        this.$message.error("Error")
                    }
                })
            },

            handle_action(e) {
                if (e.action == "update") {
                    this.actionstatus = "update"
                    this.$nextTick(() => {
                        this.form.id = e.id;
                        this.form.name = e.name;
                        this.form.desc = e.note;
                        this.form.channel = e.channel
                        this.form.content = [{
                            uid: "1",
                            name: e.name,
                            url: this.icons + '/file/' + e.content
                        }]
                        this.form.fileid = e.content
                        this.form.cover = e.cover
                    })
                    this.visible = true;
                }
                else if (e.action == 'delete') {
                    reqPost("VBOARD", { prop: "deletecontent", id: e.id }).then(res => {
                        if (res.data > 0) {
                            this.$message.success("Delete Success")
                            this.handle_get();
                        }
                        else {
                            this.$message.error("Delete ERROR")
                        }
                    })
                }
                else {
                    //……
                }
            },
            playervideo(e) {
                this.videoshow = true;
                this.$nextTick(() => {
                    this.$refs.videoPlayer.options.sources = [{
                        src: this.icons + '/file/' + e.content,
                        type: "video/mp4"
                    }]
                    this.$refs.videoPlayer.options.poster = this.icons + '/file/' + e.cover + "/large"
                })


            },

            handleCancel() {
                this.videoshow = false;
            },

        },
        computed: {

            loading() {
                return this.loadcount > 0
            },
            player() {
                return this.$refs.videoPlayer.player
            },
            dialogwidth() {
                return (document.documentElement.clientWidth / 3) * 2;
            },
            modalWidth() {
                return (document.documentElement.clientWidth);
            },
            modalHeight() {
                return (document.documentElement.clientHeight) - 200;
            },
            formItems() {
                var arr = [
                    { key: 'channel', title: this.$t('channel'), type: 'select', options: this.channels },
                    { key: 'name', type: 'input' },
                    { key: 'desc', type: 'textarea', title: this.$t('description'), rows: 3 },
                    {
                        key: 'content', type: 'upload', title: this.$t('content'),
                        accept: ".mp4,.avi,.wmv,.mpg,.mpeg,mov,.rm,.ram"
                    },
                    { key: 'cover', type: 'imglist', title: this.$t('cover'), },
                ]
                return arr;

            },
            mockData() {
                var arr = [];
                this.videos.map((e, i) => {
                    arr.push({
                        key: i.toString(),
                        title: e.name || "",
                        description: e.desc,
                        chosen: false,
                    })
                })
                return arr;
            },

            hours() {
                var arr = [];
                for (var i = 0; i < 24; i++) {
                    if (i < 10) {
                        arr.push("0" + i)
                    }
                    else {
                        arr.push(i)
                    }
                }
                return arr
            },
            mins() {
                var arr = [];
                for (var i = 0; i < 60; i++) {
                    if (i < 10) {
                        arr.push("0" + i)
                    }
                    else {
                        arr.push(i)
                    }
                }
                return arr
            },
            searchform() {
                var arr = [{
                    key: 'channel',
                    type: 'select',
                    options: this.channels,
                }];
                return arr;
            },
        },
        beforeDestroy() {
            if (this.timer) {
                clearInterval(this.timer);//在vue实例销毁钱，清除我们的定时器
            }
        }
    }
</script>

<style lang="less" scoped>
    .video-player {}


    .v-enter-active,
    .v-leave-active {
        transition: opacity 0.8s ease;
    }

    .v-enter-from,
    .v-leave-to {
        opacity: 0;
    }

    .success,
    .success:hover,
    .success:visited {
        background-color: @success-color;
        color: white;
    }

    .divclas {
        overflow-y: scroll;
        height: 200px;

    }
</style>