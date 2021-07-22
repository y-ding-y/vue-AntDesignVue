<template>
    <div class="chose_div" @click="handle_click" :id="id">
        <a-card v-bind:class="theme.mode=='night'?cardsum:cardsum2">
            <div v-bind:class="theme.mode=='night'?titletxt:titletxt2">{{title}}</div>
            <div v-if="id!='year_ship'" v-bind:class="theme.mode=='night'?totalsum:totalsum2">{{sum}}</div>
            <div v-else v-bind:class="theme.mode=='night'?totalsum:totalsum2"
                v-bind:style="{color:theme.mode=='night'?'#00F5FF':'#009ACD'  }">{{sum}}
            </div>
            <div v-bind:class="theme.mode=='night'?sumprocess:sumprocess2"
                v-bind:style="{visibility:id=='year_ship'?'hidden':'visible' }">
                <a-progress :percent="progress" status="active" size="small" :show-info="false" />
            </div>
        </a-card>
    </div>
</template>

<script>
    import bus from './../bus.js'
    import { mapState } from 'vuex'

    export default {
        props: {
            title: String,
            sum: Number,
            progress: Number,
            id: String,
            styles: Object,
            actived: { type: Array, default: () => ([]) },
        },
        data() {
            return {
                cardsum: "cardsum",
                cardsum2: "cardsum2",
                titletxt: "titletxt",
                titletxt2: "titletxt2",
                totalsum: "totalsum",
                totalsum2: "totalsum2",
                sumprocess: "sumprocess",
                sumprocess2: "sumprocess2",
            }
        },
        computed: {
            ...mapState('setting', ['theme']),
        },
        watch: {

        },
        methods: {
            handle_click() {
                bus.$emit("actived", this.id)

            }
        },
    }
</script>

<style lang="less" scoped>
    .chose_div :hover {
        cursor: pointer;
    }

    .cardsum {
        background: rgba(57, 57, 117, 0.452);
        border: 0px solid;
        /* color: rgba(224, 224, 224, 0.747); */
    }


    .titletxt {
        color: white;
        font-size: 20px;
        font-weight: 500;
    }

    .totalsum {
        color: rgb(89, 219, 95);
        font-size: 30px;
        font-weight: 999;
    }

    .sumprocess {
        color: white
    }

    /*************************************************************/
    .cardsum2 {
        background: white;
        border: 0px solid;
        /* color: rgba(224, 224, 224, 0.747); */
    }

    .titletxt2 {
        color: rgb(0, 0, 0);
        font-size: 20px;
        font-weight: 500;
    }

    .totalsum2 {
        color: rgb(63, 151, 71);
        font-size: 30px;
        font-weight: 999;
    }

    .sumprocess2 {
        color: rgb(0, 0, 0)
    }
</style>
<style lang="less" scoped>

</style>