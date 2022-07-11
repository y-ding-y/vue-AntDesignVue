<template>
    <div :class="'animated '+ animated" :style="style">
        <v-chart :onClick="click_interval" v-if="datebind " :key="i" :force-fit="true" :height="ss.height"
            :padding="padding" :width="ss.width" :data="result" :scale="scale">
            <v-tooltip />
            <v-axis v-for="(s, i) in axis" :key="'axis' + i" v-bind="s" />
            <v-legend v-if="legend.show" :position="legend.position" />
            <v-series v-for="(s, i) in series" :key="'series'+i" v-bind="s" />
        </v-chart>

        <v-chart v-if="!datebind " :key="i" :force-fit="true" :height="ss.height" :padding="padding" :width="ss.width"
            :data="data" :scale=" [{
                dataKey: 'month',
                min: 0,
                max: 1,
              }]">
            <v-tooltip />
            <v-axis v-for="(s, i) in exampleaxis" :key="'exampleaxis' + i" v-bind="s" />
            <v-legend />
            <v-line position="month*temperature" color="city" />
            <v-line position="month*rate" />
            <v-point position="month*temperature" color="city" :size="4" :v-style="stylea" :shape="'circle'" />
        </v-chart>

    </div>
</template>

<script>
    const DataSet = require('@antv/data-set');
    const sourceData = [
        { month: 'Jan', Tokyo: 7.0, London: 3.9, },
        { month: 'Feb', Tokyo: 6.9, London: 4.2, },
        { month: 'Mar', Tokyo: 9.5, London: 5.7, },
        { month: 'Apr', Tokyo: 14.5, London: 8.5, },
        { month: 'May', Tokyo: 18.4, London: 11.9, },
        { month: 'Jun', Tokyo: 21.5, London: 15.2, },
        { month: 'Jul', Tokyo: 25.2, London: 17.0, },
        { month: 'Aug', Tokyo: 26.5, London: 16.6, },
        { month: 'Sep', Tokyo: 23.3, London: 14.2, },
        { month: 'Oct', Tokyo: 18.3, London: 10.3, },
        { month: 'Nov', Tokyo: 13.9, London: 6.6, },
        { month: 'Dec', Tokyo: 9.6, London: 4.8, },
    ];

    const dv = new DataSet.View().source(sourceData);
    dv.transform({
        type: 'fold',
        fields: ['Tokyo', 'London'],
        key: 'city',
        value: 'temperature',
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
                toroute: null,
                // legendStyle: {
                //     MARKER_CLASS: {
                //         width: '68px',
                //         height: '68px',
                //         lineHeight: '68px',
                //         borderRadius: '50px',
                //         display: 'inline-block',
                //         marginRight: '4px',
                //         textAlign: 'center',
                //         fontZize: '10px',
                //         marginTop: '3px',
                //         color: 'white',
                //         float: 'left',
                //         borderStyle: 'solid',
                //         borderWidth: '1px',
                //         borderColor: 'red'
                //     }
                // },
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
            ss() {
                this.i++
            },
        },
        computed: {
            datarefresh() {
                return this.ss.refreshdata
            },
            exampleaxis() {
                return [{
                    dataKey: 'temperature',
                    show: true,
                    label: {
                        textStyle: {
                            fill: this.ss.color,
                            fontWeight: this.ss.fontweight,
                            fontSize: this.ss.fontSize,
                        },
                    },
                },
                {
                    dataKey: 'month',
                    show: true,
                    label: {
                        textStyle: {
                            fill: this.ss.color,
                            fontWeight: this.ss.fontweight,
                            fontSize: this.ss.fontSize,
                        },
                    },
                },]
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
            scale() {
                var arr = [];
                if (this.ss.AxisY1.isshow) {
                    arr.push({
                        dataKey: this.ss.AxisY1.name,
                        min: 0,
                    })
                }
                if (this.ss.AxisY2.isshow) {
                    arr.push({
                        dataKey: this.ss.AxisY2.name,
                        min: 0,
                    })

                }
                return arr
            },
            series() {
                var arr = [];
                if (this.ss.AxisY1.isshow) {
                    arr = [...arr, ...this.switchcharttype(this.ss.AxisY1)]
                }
                if (this.ss.AxisY2.isshow) {
                    arr = [...arr, ...this.switchcharttype(this.ss.AxisY2)]
                }
                return arr
            },

            legend() {
                return {
                    show: this.ss.islegend,
                    position: "top-right",
                    // itemTpl: (value, color, checked, index) => {
                    //     console.log(index)
                    //     var markerDom = '<span class="g2-legend-marker" style="background-color:' + color + '"></span>';
                    //     var nameDom = '<span class="legend-item-name" style="color:' + this.ss.color + '">' + value + '</span>';
                    //     return '<div class="g2-legend-list-item" style="display:flex" data-value="' + value + '">' + markerDom + nameDom + '</div>';

                    // },

                }

            },
            axis() {
                return [{
                    dataKey: this.ss.AxisX.name,
                    show: this.ss.AxisX.isshow,
                    label: {
                        textStyle: {
                            fill: this.ss.color,
                            fontWeight: this.ss.fontweight,
                            fontSize: this.ss.fontSize,
                        },
                    },
                },
                {
                    dataKey: this.ss.AxisY1.name,
                    show: this.ss.AxisY1.isshow,
                    label: {
                        textStyle: {
                            fill: this.ss.color,
                            fontWeight: this.ss.fontweight,
                            fontSize: this.ss.fontSize,
                        },
                    },
                },
                {
                    dataKey: this.ss.AxisY2.name,
                    show: this.ss.AxisY2.isshow,
                    label: {
                        textStyle: {
                            fill: this.ss.color,
                            fontWeight: this.ss.fontweight,
                            fontSize: this.ss.fontSize,
                        },
                    },
                },]
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
                }
            }
        },
        methods: {
            click_interval(e) {
                if (e != undefined && e.data != undefined && this.toroute != null) {
                    let temp = eval("(" + this.toroute + ")")
                    window.open(this.$router.resolve({
                        path: temp.path
                    }).href, temp.type)
                }
            },
            switchcharttype(e) {
                var res = [];
                switch (e.charttype) {
                    case "bar":
                        res = [{
                            quickType: "bar",
                            position: this.ss.AxisX.name + '*' + e.name,
                            color: e.chartkey,
                            adjust: [
                                {
                                    type: 'dodge',
                                    marginRatio: 1 / 32,
                                },
                            ],
                        }]
                        break;
                    case "point-line":
                        res = [
                            {
                                quickType: 'line',
                                position: this.ss.AxisX.name + '*' + e.name,
                                color: e.chartkey,
                            },

                            {
                                quickType: 'point',
                                position: this.ss.AxisX.name + '*' + e.name,
                                color: e.chartkey,
                            }
                        ]
                        break;
                    default:
                        res = [];
                        break;
                }
                return res
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
