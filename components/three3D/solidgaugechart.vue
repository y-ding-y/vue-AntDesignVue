<template>
    <div :class="'animated '+ animated" :style="style">
        <div v-if="curdiv" :style="[styleclick]"> </div>
        <highcharts :options="container" :key="i"></highcharts>
        <!-- :callback="myCallback" -->
    </div>
</template>

<script>

    const DataSet = require('@antv/data-set');
    const data = { y: 0 };
    export default {
        props: {
            ss: {
                type: Object, default: () => ({
                    left: 0,
                    width: 100,
                    height: 100,
                })
            },
            curdiv: { type: Boolean, default: false },
        },
        data() {
            return {
                isshow: true,
                i: 0,
                result: data,
                data,
                animated: this.ss.inanimate,
                refreshData: null,

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
                    this.result = this.data
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
            // chart() {
            //     this.i++
            // },
        },
        computed: {
            credits() {
                return this.ss.highchart.credits
            },
            chart() {
                return this.ss.highchart.chart
            },
            plotOptions() {
                return this.ss.highchart.plotOptions
            },
            yAxis() {
                return this.ss.highchart.yAxis
            },
            series() {
                return this.ss.highchart.series
            },
            pane() {
                return this.ss.highchart.pane
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

            style() {
                return {
                    // border: this.curdiv ? '5px solid blue' : '0px',
                    position: 'absolute',
                    top: this.ss.top + 'px',
                    left: this.ss.left + 'px',
                    width: this.ss.width + "px",
                    height: this.ss.height + "px",
                    backgroundColor: 'rgba(0,0,0,0)',
                    zIndex: this.ss.zindex
                }
            },
            styleclick() {
                return {
                    border: this.curdiv ? '5px solid blue' : '0px',
                    position: 'absolute',
                    width: this.ss.width + "px",
                    height: this.ss.height + "px",
                }
            },
            container() {
                return {
                    credits: {
                        enabled: this.credits.enabled,
                        text: this.credits.text,
                        href: null,
                        position: {
                            align: this.credits.align,
                            verticalAlign: this.credits.verticalalign,
                            x: this.credits.positionx,
                            y: this.credits.positiony,
                        },
                        style: {
                            color: this.credits.color,
                            fontSize: this.credits.fontsize,
                            fontWeight: this.credits.fontweight
                        }
                    },
                    chart: {
                        type: "solidgauge",
                        height: this.ss.height,
                        width: this.ss.width,
                        backgroundColor: null,
                        spacingTop: this.chart.spacingtop,
                        spacingRight: this.chart.spacingright,
                        spacingBottom: this.chart.spacingbottom,
                        spacingLeft: this.chart.spacingleft,
                    },
                    pane: {
                        center: [this.pane.center[0] + '%', this.pane.center[1] + '%'],
                        size: this.pane.size + '%',
                        startAngle: this.pane.startangle,
                        endAngle: this.pane.endangle,
                        background: {
                            backgroundColor: this.pane.backgroundcolor,
                            borderColor: this.pane.bordercolor,
                            borderWidth: this.pane.borderwidth,
                            innerRadius: this.pane.innerradius + '%',
                            outerRadius: this.pane.outerradius + '%',
                            shape: this.pane.shape
                        }
                    },
                    title: {
                        text: null
                    },
                    plotOptions: {
                        series: {
                            states: {
                                inactive: {
                                    enabled: false
                                }
                            },
                            dataLabels: {
                                enabled: this.plotOptions.series.dataLabels.enabled,
                                format: '{y}',
                                style: {
                                    fontSize: this.plotOptions.series.dataLabels.style.fontsize,
                                    color: this.plotOptions.series.dataLabels.style.color,
                                    textOutline: this.plotOptions.series.dataLabels.style.border + "px   "
                                        + this.plotOptions.series.dataLabels.style.bordercolor,
                                    fontWeight: this.plotOptions.series.dataLabels.style.fontweight,
                                },
                                align: this.plotOptions.series.dataLabels.align,
                                verticalAlign: this.plotOptions.series.dataLabels.verticalalign,
                                x: this.plotOptions.series.dataLabels.x,
                                y: this.plotOptions.series.dataLabels.y,
                                borderWidth: 0
                            },
                        },
                    },

                    yAxis: this.yAxis.map(e => {
                        return {
                            offset: e.offset,
                            visible: e.enabled,
                            min: e.min,
                            max: e.max,
                            tickAmount: e.tickcount,
                            lineWidth: e.linewidth,
                            lineColor: e.linecolor,
                            tickWidth: e.tickwidth,
                            tickColor: e.tickcolor,
                            tickLength: e.ticklength,
                            tickPosition: e.tickposition,
                            // stops: [
                            //     [0.1, '#55BF3B'], // green
                            //     [0.5, '#DDDF0D'], // yellow
                            //     [0.9, '#DF5353'] // red
                            // ],
                            labels: { //   format: '{value} mm',
                                style: {
                                    fontSize: e.labels.style.fontsize,
                                    color: e.labels.style.color,
                                    textOutline: e.labels.style.border + "px   "
                                        + e.labels.style.bordercolor,
                                    fontWeight: e.labels.style.fontweight,
                                },
                                distance: e.distance
                            },
                            title: {
                                text: e.titletext,
                                align: e.titlealign,
                                rotation: e.rotation,
                                x: e.offsetx,
                                y: e.offsety,
                                style: {
                                    fontSize: e.labels.style.fontsize,
                                    color: e.labels.style.color,
                                    textOutline: e.labels.style.border + "px   "
                                        + e.labels.style.bordercolor,
                                    fontWeight: e.labels.style.fontweight,
                                },
                            },
                        }
                    }),
                    series: this.series.map(s => {
                        return {
                            type: s.type,
                            yAxis: s.yAxis,
                            dial: {
                                radius: s.dialradius + '%',
                                backgroundColor: s.dialbackgroundcolor,
                                borderColor: s.dialbordercolor,
                                borderWidth: s.dialborderwidth,
                                baseWidth: s.dialbasewidth,
                                topWidth: s.dialtopwidth,
                                baseLength: s.diallbaselength + '%', // of radius
                                rearLength: s.dialrearlength + '%'
                            },
                            data: [
                                {
                                    color: s.color,
                                    radius: s.radius + '%',
                                    innerRadius: s.innerradius + '%',
                                    y: Number(this.result[s.yname])

                                }],
                            dataLabels: {
                                enabled: true,
                                format: '{y}',
                                style: {
                                    fontSize: s.fontsize,
                                    color: s.fontcolor,
                                    textOutline: s.border + "px   "
                                        + s.bordercolor,
                                    fontWeight: s.fontweight,
                                },
                                align: s.align,
                                verticalAlign: s.verticalalign,
                                x: s.x,
                                y: s.y,
                                borderWidth: 0,
                            },
                        }
                    })
                }
            },
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
            handle_search() {
                //let dtype = this.ss.datetype;
                let dsource = this.ss.datesource
                switch (dsource) {
                    case "phpapi":
                        this.handle_phpapi(dsource);
                        break;
                    case "phpsocket":
                        this.handle_phpsocket(dsource);
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

            handle_phpsocket(dsource, dtype) {
                let temp = eval("(" + this.ss[dsource] + ")")
                console.log(temp, dtype)

                this.$socket.emit('threeD', { "asd": "asd" });

                if (temp.url != '') {
                    console.log(this.$socket.connected)
                    this.$socket.emit("threeD", temp)

                    // console.log(temp.params.prop)
                    // this.$socket.on("chatserver", msg => {
                    //     console.log(msg)
                    // })
                }

                // this.ss[dsource], data => {
                //     this.temp_nodejsemit = this.ss[dsource]
                //     if (typeof (data) == dtype) {
                //         this.result = data
                //     }
                //     else {
                //         this.$message.error(this.ss.componentname + "数据格式错误")
                //     }
                // })
            },

            // myCallback(e) {
            //     this.refreshData = setInterval(() => {
            //         var point = e.series[0].points[0];
            //         var inc = Math.round((Math.random() - 0.5) * 100);
            //         var newVal = point.y + inc;

            //         if (newVal < 0 || newVal > 200) {
            //             newVal = point.y - inc;
            //         }
            //         point.update(Number(newVal));
            //     }, Number(this.ss.refreshdatatime) * 1000 * 60);


            // },
        }
    }
</script>
