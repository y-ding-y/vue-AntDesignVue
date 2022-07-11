<template>
    <div :class="'animated '+ animated" :style="style">
        <v-chart :onClick="click_interval" :key="i" v-if="datebind" :forceFit="true" :height="ss.height" :data="result"
            :padding="padding">
            <v-tooltip :showTitle="false" dataKey="defectname*percent" />
            <v-axis />
            <v-pie :position="ss.piechart ? ss.piechart.position:'qty'" :color="ss.piechart ? ss.piechart.color:'name'"
                :vStyle="pieStyle" :label="labelConfig" />
            <v-coord type="theta" :radius="ss.radius" :innerRadius="ss.innerradius" />
        </v-chart>

        <v-chart v-if="!datebind" :forceFit="true" :height="height" :data="data" :scale="[{
            dataKey: 'percent',
            min: 0,
            formatter: '.0%',
        }]">
            <v-tooltip :showTitle="false" dataKey="item*percent" />
            <v-axis />
            <v-legend dataKey="item" />
            <v-pie position="percent" color="item" :vStyle="pieStyle" :label="labelConfig" />
            <v-coord type="theta" :radius="0.5" :innerRadius="0.6" />
        </v-chart>

    </div>
</template>

<script>
    const DataSet = require('@antv/data-set');

    const sourceData = [
        { item: '事例一', count: 40 },
        { item: '事例二', count: 21 },
        { item: '事例三', count: 17 },
        { item: '事例四', count: 13 },
        { item: '事例五', count: 9 }
    ];

    const dv = new DataSet.View().source(sourceData);
    dv.transform({
        type: 'percent',
        field: 'count',
        dimension: 'item',
        as: 'percent'
    });
    const data = dv.rows;

    export default {
        props: {
            ss: {
                type: Object, default: () => ({
                    left: 0,
                    width: 100,
                    height: 100,
                })
            },
            curdiv: { type: Boolean, default: false }
        },
        data() {
            return {
                isshow: true,
                data,
                stylea: { stroke: '#fff', lineWidth: 1 },
                //padding: [48,48,48,48]
                i: 0,
                animateclass: "",
                result: data,
                animated: this.ss.inanimate,

                height: 400,
                refreshData: null,

                pieStyle: {
                    stroke: '#fff',
                    lineWidth: 1,
                },
                toroute: null,
            }
        },
        created() {
            if (this.datebind) {
                if (this.datarefresh) {
                    this.refreshData = setInterval(() => {
                        this.handle_search()
                    }, Number(this.ss.refreshdatatime) * 1000 * 60);
                }
                else {
                    this.handle_search()
                }
            }
            this.isshow = this.ss.isshow
            if (this.ss.isshow) {
                this.animated = this.ss.inanimate
            }
            else {
                this.animated = this.ss.outanimate
            }
        },
        beforeDestroy() {
            clearInterval(this.refreshData);
            this.refreshData = null;
        },
        watch: {
            isshow(vnew, vold) {
                if (this.isshow) {
                    this.animated = this.ss.inanimate
                }
                else if (vnew != vold && !this.isshow) {
                    this.animated = this.ss.outanimate
                }
            },
            datebind(vnew, vold) {
                if (this.datebind && vnew != vold) {
                    this.handle_search();
                }
                else {
                    this.result = data
                }
            },
            datedetail() {
                this.handle_search();
            },
            datesource() {
                this.handle_search();
            },
            datarefresh() {
                if (this.datarefresh) {
                    this.refreshData = setInterval(() => {
                        this.handle_search()
                    }, Number(this.ss.refreshdatatime) * 1000 * 60);
                } else {
                    clearInterval(this.refreshData);
                    this.refreshData = null;
                }

            },
            ss() {
                this.i++
            },
        },
        computed: {
            labelConfig() {
                return [this.ss.piechart ? this.ss.piechart.position : 'qty', {
                    formatter: (val, item) => {
                        return item.point[this.ss.piechart ? this.ss.piechart.color : 'name'] + ':\n\r' + val //':\n\r' + 
                    },
                    label: {
                        textStyle: {
                            fontSize: this.ss.fontSize,
                            fill: this.ss.color,
                            fontWeight: this.ss.fontweight
                        },
                    },
                }]
            },
            datarefresh() {
                return this.ss.refreshdata
            },
            datebind() {
                return this.ss.datebind
            },
            datesource() {
                return this.ss.datesource
            },
            datedetail() {
                return this.ss[this.ss.datesource]
            },


            padding() {
                if (this.ss.paddingleft == undefined) {
                    return [24, 24, 24, 24]
                }
                else {
                    return [
                        this.ss.paddingtop,
                        this.ss.paddingright,
                        this.ss.paddingbottom,
                        this.ss.paddingleft
                    ]
                }

            },
            style() {
                return {
                    border: this.curdiv ? '5px solid blue' : '0px',
                    position: 'absolute',
                    top: this.ss.top + 'px',
                    left: this.ss.left + 'px',
                    width: this.ss.width + "px",
                    height: this.ss.height + "px",
                    backgroundColor: 'rgba(0,0,0,0)',
                    zIndex: this.ss.zindex
                }
            }
        },
        methods: {
            click_interval(e) {
                // var temp = this.$route.fullPath.split('/')
                // this.$router.push("../../../smt/product/yield")

                console.log(this.toroute)
                if (e != undefined && e.data != undefined && this.toroute != null) {
                    let temp = eval("(" + this.toroute + ")")
                    window.open(this.$router.resolve({
                        path: temp.path
                    }).href, temp.type)
                }
            },
            handle_search() {
                //let dtype = this.ss.datetype;
                let dsource = this.ss.datesource
                switch (dsource) {
                    case "phpapi":
                        this.handle_phpapi(dsource);
                        break;
                    case "nodejsemit":
                        this.handle_nodejsemit(dsource);
                        break;
                    default:
                        break;
                }

            },

            handle_phpapi(dsource) {
                let temp = eval("(" + this.ss[dsource] + ")")
                if (temp.url != '') {
                    this.$funAxios.reqPost(temp.url, temp.params).then(res => {
                        if (this.ss.transform) {
                            const dv = new DataSet.View().source(res.data)
                            dv.transform(eval("(" + this.ss.transformjson + ")"))
                            this.result = dv.rows
                        }
                        else {
                            this.result = res.data;
                        }
                    })
                        .catch(error => {
                            console.log(error)
                            this.result = []
                            this.$message.error(this.ss.componentname + "NetWork Error")
                        })
                }
            },

            handle_nodejsemit(dsource) {
                this.sockets.unsubscribe(this.temp_nodejsemit);
                this.sockets.subscribe(this.ss[dsource], data => {
                    this.temp_nodejsemit = this.ss[dsource]
                    this.result = data

                })
            },
        }
    }
</script>
