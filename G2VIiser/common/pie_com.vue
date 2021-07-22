<template>
    <div>
        <a-card v-bind:class="theme.mode=='night'?cardsum:cardsum2">
            <h4 v-if="title" v-bind:style="theme.mode=='night'?titleStyle:titleStyle2">{{ title }}
            </h4>
            <v-chart :forceFit="true" :height="height" :data="data" :scale="[
                  {
                    dataKey: 'percent',
                    formatter: '.0%',
                  },
                ]" :padding="padding">
                <v-tooltip :showTitle="false" dataKey="name*qty" />
                <v-axis />
                <v-legend dataKey="name" v-if="false" />
                <v-pie position="qty" color="name" :vStyle="pieStyle" :label="labelConfig" />
                <v-coord type="theta" :radius="0.55" :innerRadius="0.4" />
            </v-chart>
        </a-card>
    </div>
</template>

<script>
    import { mapState } from 'vuex'
    export default {
        props: {
            title: String,
            data: Array,
            height: Number,
            padding: { type: Array, default: () => [44, 24, 4, 44] }
        },
        data() {
            return {
                cardsum: "cardsum",
                cardsum2: "cardsum2",
                pieStyle: {
                    stroke: '#fff',
                    lineWidth: 0.5,
                },
                titleStyle: { textAlign: "center", color: "white", fontSize: "20px" },
                titleStyle2: { textAlign: "center", color: "black", fontSize: "20px" },

            }
        },
        computed: {
            ...mapState('setting', ['theme']),
            labelConfig() {
                return [
                    'percent',
                    {
                        formatter: (val, item) => {
                            return item.point.name + ':' + val
                        },
                        label: {
                            textStyle: {
                                fontSize: 14,
                                fill: this.theme.mode == 'night' ? "white" : "black",
                                fontWeight: 400
                            },
                        },

                    },
                ]
            },
        },
        components: {

        },
    }
</script>

<style lang="less" scoped>
    .cardsum {
        background: rgba(80, 80, 110, 0.452);
        border: 0px solid;
    }

    .cardsum2 {
        background: rgb(255, 255, 255);
        border: 0px solid;
    }
</style>