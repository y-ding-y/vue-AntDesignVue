<template>
    <div :style="demoInfiniteContainer" v-infinite-scroll="loadMore" infinite-scroll-disabled="busy"
        infinite-scroll-distance="10">
        <a-list item-layout="vertical" size="large" :data-source="videos">
            <!-- :pagination="pagination" -->
            <a-list-item slot="renderItem" key="item.name" slot-scope="item">
                <template v-for="{ type, text,action } in actions" slot="actions">
                    <span :key="type" @click="handle_action(item,action)">
                        <a-icon :type="type" style="margin-right: 8px" />
                        {{ text }}
                    </span>
                </template>

                <img slot="extra" height="110" width="170" alt="logo" :src="item.coverurl"
                    @click="handle_video(item)" />
                <!-- <video slot="extra" width="200" :src="item.contenturl" controls="controls"></video> -->
                <a-list-item-meta>
                    <a slot="title" @click="handle_video(item)">{{ item.name }}</a>
                </a-list-item-meta>
                {{ item.desc }}
            </a-list-item>
        </a-list>
    </div>


</template>
<script>

    import { mapState } from 'vuex'
    export default {
        props: {
            videos: Array,
        },
        data() {
            return {
                busy: false,
                pagination: {
                    onChange: page => {
                        console.log(page);
                    },
                    pageSize: 4,
                },
                actions: [
                    // { type: 'star-o', text: '156' },
                    { type: 'delete', text: '删除', action: "delete" },
                    { type: 'edit', text: '修改', action: "update" },
                ],
            };
        },
        computed: {
            ...mapState('setting', ['lang', 'pageMinHeight']),
            demoInfiniteContainer() {
                return {
                    ' border': '1px solid #e8e8e8',
                    'borderRadius': '4px',
                    'overflowY': 'scroll',
                    'padding': '8px 24px',
                    'height': Number(this.pageMinHeight - 100) + 'px',
                }
            },
        },
        methods: {
            handle_action(e, action) { 
                this.$emit("action", { ...e, ...{ action: action } })
            },
            handle_video(e) {
                this.$emit("playervideo", e)
            },
            loadMore() {
                // this.busy = true
                // for (var i = 0, j = this.videos.length; i < j; i++) {
                //     this.videos.push({ name: this.count++ })
                // }
                // console.log(this.videos)
                // this.busy = false

            }
            // handleInfiniteOnLoad() {
            //     const data = this.videos;
            //     this.loading = true;
            //     if (data.length > 5) {
            //         this.$message.warning('Infinite List loaded all');
            //         this.busy = true;
            //         this.loading = false;
            //         return;
            //     }
            //     // this.fetchData(res => {
            //     //     this.data = data.concat(res.results);
            //     //     this.loading = false;
            //     // });
            // },
        }
    };
</script>

<style lang="less">
    .demo-infinite-container {
        border: 1px solid #e8e8e8;
        border-radius: 4px;
        overflow: auto;
        padding: 8px 24px;
        height: 500px;
    }

    .demo-loading-container {
        position: absolute;
        bottom: 40px;
        width: 100%;
        text-align: center;
    }

    img:hover {
        cursor: pointer
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
</style>