<template>
    <ef-main :loading="loading" :isMainPage="true" @adds="show_weather" :adds="true" :group="true" @group="group_edit"
        :weather="true">

        <v-list @playervideo="playervideo" @action="handle_action" :videos="videos"></v-list>

        <a-drawer :width="dialogwidth" placement="right" :closable="false" :visible="visible" @close="onClose">
            <v-form ref="formchild" title="上传视频" :form-items="formItems" @submit="submit" :rules="rules" :form="form">
            </v-form>
        </a-drawer>

        <a-drawer :width="dialogwidth" placement="right" :closable="false" :visible="groupvisible" @close="groupClose">
            <a-form-model :label-col="{ span: 5 }" :wrapper-col="{ span: 19 }" :key="groupi">
                <a-form-model-item label="已有分組">
                    <div class="divclas">
                        <a-button style="width: 100%;" @click="handle_search">全部视频</a-button>
                        <a-button style="width: 50%;" v-for="(m,i) in groups" :key="i" @click="showgroup(m.val)">
                            {{m.name}}-{{m.times}}
                        </a-button>

                    </div>
                </a-form-model-item>

                <a-form-model-item label="分組">
                    <a-transfer :data-source="mockData" :titles="['Source', 'Target']" :list-style="{
                        width: '40%',
                        height: '300px',
                      }" :target-keys="targetKeys" :render="renderItem" @change="handleChange" />

                </a-form-model-item>
                <a-form-model-item label="分組名">
                    <a-input v-model="groupname"></a-input>
                </a-form-model-item>
                <a-form-model-item label=" 播放時間 ">
                    <a-select v-model="hour" style="width: 60px">
                        <a-select-option v-for="(item,index) in hours" :key="'hour'+index" :value="item">
                            {{item}}
                        </a-select-option>
                    </a-select>
                    時
                    <a-select v-model="min" style="width: 60px">
                        <a-select-option v-for="(item,index) in mins" :key="'min'+index" :value="item">
                            {{item}}
                        </a-select-option>
                    </a-select>
                    分
                </a-form-model-item>
                <a-button :class="'success'" style="width: 100%;" @click="handle_addgroup">提交
                </a-button>
            </a-form-model>

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


    const i18n = require('./i18n')
    const rules = {
        name: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
        desc: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
        cover: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
        content: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
    }

    const form = {
        name: "",
        desc: "",
        cover: [],
        content: [],
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
            }
        },

        components: {
            videoPlayer, EfMain, VList, VForm
        },
        created() {
            this.handle_search();
            this.handle_searchgroup();
        },

        mounted() {

        },
        methods: {
            // onPlayerPlay(player) {
            //     console.log(player)
            // }, 
            handle_search() {
                this.groupvisible = false;
                let filedata = Array(this.$localForage.getItem('videos'))
                Promise.all(filedata).then(value => {
                    if (value[0] != null) {
                        this.videos = value[0].map((e, i) => {
                            e.id = i;
                            e.coverurl = URL.createObjectURL(e.cover)
                            e.contenturl = URL.createObjectURL(e.content)
                            return e;
                        })
                    }
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
            //方法
            full(element) {
                //做兼容处理 
                if (element.requestFullscreen) {
                    element.requestFullscreen();
                } else if (element.mozRequestFullScreen) {
                    element.mozRequestFullScreen();
                } else if (element.msRequestFullscreen) {
                    element.msRequestFullscreen();
                } else if (element.oRequestFullscreen) {
                    element.oRequestFullscreen();
                } else if (element.webkitRequestFullscreen) {
                    element.webkitRequestFullScreen();
                } else {
                    var docHtml = document.documentElement;
                    var docBody = document.body;
                    var videobox = document.getElementsByClassName("video-player");
                    var cssText = "width:100%;height:100%;overflow:hidden;";
                    docHtml.style.cssText = cssText;
                    docBody.style.cssText = cssText;
                    videobox.style.cssText = cssText + ";" + "margin:0px;padding:0px;";
                    document.IsFullScreen = true;
                }
            },
            playerReadied(e) {
                console.log(e)
                // const myplayer = this.$refs.videoPlayer.player;
                // this.full(myplayer)
            },
            onPlayerPlay() {
                // const myplayer = this.$refs.videoPlayer.player;
                // //   this.full(myplayer)
                // if (!myplayer.isFullscreen()) {
                //     myplayer.requestFullscreen()
                //     myplayer.isFullscreen(true)
                // }
            },
            playerend() {
                // this.videoshow = false

            },
            submit(value) {
                if (this.actionstatus == "add") {
                    this.videos.push({
                        name: value.name,
                        desc: value.desc,
                        cover: new Blob([value.cover[0].originFileObj], { type: value.cover[0].type }),
                        content: new Blob([value.content[0].originFileObj], { type: value.content[0].type }),
                    })
                }
                else if (this.actionstatus == "update") {
                    var temp = {
                        name: value.name,
                        desc: value.desc,
                        cover: value.cover[0].originFileObj ?
                            new Blob([value.cover[0].originFileObj], { type: value.cover[0].type }) :
                            this.form.cover[0].val,
                        content: value.content[0].originFileObj ?
                            new Blob([value.content[0].originFileObj], { type: value.content[0].type }) :
                            this.form.content[0].val,
                    }
                    this.videos.splice(this.form.id, 1, temp)
                }
                var that = this;
                this.$localForage.setItem('videos', this.videos).then(function () {
                    that.visible = false;
                    that.handle_search();
                    that.$refs.formchild.clearForm();
                })
                this.visible = false;
            },
            handle_action(e) {
                if (e.action == "update") {
                    this.actionstatus = "update"
                    this.$nextTick(() => {
                        this.form.id = e.id;
                        this.form.name = e.name;
                        this.form.desc = e.desc;
                        this.form.cover = [{ uid: 1, name: e.name, url: URL.createObjectURL(e.cover), val: e.cover }]
                        this.form.content = [{ uid: 2, name: e.name, url: URL.createObjectURL(e.content), val: e.content }];
                    })
                    this.visible = true;
                }
                else {
                    this.videos.splice(e.id, 1)
                    var that = this;
                    this.$localForage.setItem('videos', this.videos).then(function () {
                        that.handle_search();
                        that.$message.success("Delete Success")
                    })
                }
            },
            playervideo(e) {
                this.videoshow = true;
                this.$nextTick(() => {
                    // let video = document.createElement('video')
                    // video.src = e.contenturl
                    // video.addEventListener('loadedmetadata', function (e2) {
                    //     console.log(video.duration)
                    //     console.log(e2)
                    // }) 
                    this.$refs.videoPlayer.options.sources = [{
                        src: e.contenturl,
                        type: "video/mp4"
                    }]
                    this.$refs.videoPlayer.options.poster = e.coverurl
                })


            },
            showModal() {
                this.visible = true;
            },

            handleCancel() {
                this.videoshow = false;
            },
            group_edit() {
                this.groupvisible = true;
            },

            renderItem(item) {
                const customLabel = (
                    <span class="custom-item">
                        {item.title} - {item.description}
                    </span>
                );
                return {
                    label: customLabel, // for displayed item
                    value: item.title, // for title and filter matching
                };
            },
            handleChange(targetKeys, direction, moveKeys) {
                console.log(targetKeys, direction, moveKeys);
                this.targetKeys = targetKeys;
            },
            handle_addgroup() {
                var temp = { name: this.groupname, times: this.hour + ":" + this.min, val: [] };
                this.targetKeys.map(e => {
                    temp.val.push(this.videos[e])
                })
                this.groups.push(temp)
                var that = this
                this.loadcount = 1;
                this.$localForage.setItem('groups', this.groups).then(function () {
                    that.loadcount--
                    that.groupi++
                    that.groupvisible = false;
                })
                // this.hour = "";
                // this.min = "";
                // this.groupname = "";
                // this.targetKeys = [];

            },
            showgroup(e) {
                this.groupvisible = false
                this.videos = e.map(m => {
                    m.coverurl = URL.createObjectURL(m.cover)
                    m.contenturl = URL.createObjectURL(m.content)
                    return m;
                });
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
                    { key: 'name', type: 'input' },
                    { key: 'desc', type: 'textarea', title: this.$t('description'), rows: 3 },
                    {
                        key: 'cover', type: 'upload', title: this.$t('cover'),
                        picture: true, accept: ".jpg,.jpeg,.png,.gif"
                    },
                    {
                        key: 'content', type: 'upload', title: this.$t('content'),
                        accept: ".mp4,.avi,.wmv,.mpg,.mpeg,mov,.rm,.ram"
                    },
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
            }
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