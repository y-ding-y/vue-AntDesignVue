<template>
    <div id="components-form-demo-advanced-search">
        <a-row :gutter="0">
            <a-col v-for="item in values" :key="item.key" :span="12">
                <div class="div-class">
                    <div class="key-class" v-if="item.key!=''">{{$t(item.key)}}：</div>
                    <div class="value-class">{{item.value}}</div>
                </div>

            </a-col>
        </a-row>
    </div>
</template>
<script>
    import { merge } from '@/utils/util'

    const i18n = require('./i18n')
    export default {
        name: "snview_form",
        props: {
            actionFix: String,
            keys: Array,
            valuelist: Object
        },
        i18n: merge(require('@/i18n'), i18n),
        data() {
            return {
                expand: false,
                temps: this.keys
            };
        },

        created() {
            if (this.keys.length % 2 > 0) {
                this.temps.push("")
            }
        },
        computed: {
            values() {
                return this.temps.map(e => {
                    return {
                        key: e == "" ? "" : e,
                        value: e == "" ? "" : this.valuelist[e]
                    }
                })
            },
        },
        methods: {

        },
    };
</script>
<style lang="less">
    .div-class {
        width: 100%;
        display: flex;
        padding: 5px 0;
        border-top: 1px solid @hover-bg-color;
    }

    .key-class {
        font-weight: 999;
        text-align: right;
        width: 40%;
        word-wrap: break-word;
        padding: 4px;
    }

    .value-class {
        font-weight: 555;
        text-align: left;
        width: 60%;
        word-wrap: break-word;
        padding: 4px;
    }

    #components-form-demo-advanced-search {
        border-bottom: 1px solid @hover-bg-color;
    }
</style>