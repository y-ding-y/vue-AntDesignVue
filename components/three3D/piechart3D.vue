<template>
    <div :class="'animated '+ animated" :style="style">
        <div v-if="curdiv" :style="[styleclick]"> </div>
        <highcharts :options="container" :key="i"></highcharts>

        <v-modaldetail :modaltitle="modaltitle" :visible="visible" @closemodal="handleCancel" :params="params">
        </v-modaldetail>

    </div>
</template>

<script>
    import VModaldetail from './modaldetail.vue'

    const DataSet = require('@antv/data-set');
    const data = [
        {
            name: 'Chrome',
            y: 0.5,
            x: "as",
            sliced: true,
            color: "orange"
        },
        {
            name: "Safari",
            y: 1,
            color: "green"
        },
    ];
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
        components: {
            VModaldetail
        },
        data() {
            return {
                isshow: true,
                i: 0,
                visible: false,
                result: data,
                data,
                animated: this.ss.inanimate,
                refreshData: null,

                toroute: null,
                modaltitle: "",
                params: "{}",
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
            chart() {
                this.i++
            },
        },
        computed: { 
            credits() {
                return this.ss.highchart.credits
            },
            chart() {
                return this.ss.highchart.chart
            },
            tooltip() {
                return this.ss.highchart.tooltip
            },
            plotOptions() {
                return this.ss.highchart.plotOptions
            },
            xAxis() {
                return this.ss.highchart.xAxis
            },
            yAxis() {
                return this.ss.highchart.yAxis
            },
            series() {
                return this.ss.highchart.series
            },
            legend() {
                return this.ss.highchart.legend
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
                    //border: this.curdiv ? '5px solid blue' : '0px',
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
                var chart_frame = {};
                var arr = ['bottom', 'side', 'back']
                arr.map(e => {
                    if (this.chart.frame[e].enabled) {
                        chart_frame[e] = {
                            size: this.chart.frame[e].size,
                            color: this.chart.frame[e].color
                        }
                    }
                    else {
                        delete chart_frame[e]
                    }
                })
                const that = this
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
                        // type: 'line', 
                        height: this.ss.height,
                        width: this.ss.width,
                        options3d: {
                            fitToPlot: true,
                            enabled: this.chart.options3denabled,
                            alpha: this.chart.options3dalpha,
                            beta: this.chart.options3dbeta,
                            depth: this.chart.options3ddepth,
                            frame: chart_frame
                        },
                        backgroundColor: null,
                        spacingTop: this.chart.spacingtop,
                        spacingRight: this.chart.spacingright,
                        spacingBottom: this.chart.spacingbottom,
                        spacingLeft: this.chart.spacingleft,
                    },
                    title: {
                        text: null
                    },
                    legend: {
                        enabled: this.legend.enabled,
                        layout: this.legend.layout,
                        floating: this.legend.floating,
                        x: this.legend.x,
                        y: this.legend.y,
                        align: this.legend.align,
                        verticalAlign: this.legend.verticalalign,
                        itemStyle: {
                            color: this.legend.color,
                            fontSize: this.legend.fontsize
                        },
                        itemHoverStyle: {
                            color: this.legend.hovercolor
                        }
                    },
                    tooltip: {
                        formatter: function () {
                            if (this.points != undefined) {
                                var res = ""
                                var title = "";
                                this.points.map(e => {
                                    title = e.key
                                    res = res + "<span style=\"color:" + e.point.color + "\" >" + e.series.name + "</span>：" + e.point.y + "<br>"
                                })
                                return title + "<br>" + res
                            }
                            else {
                                return this.point.name + '<br><b>' + this.point.y
                            }
                        },
                        shared: true,
                        // format: "{point.y}",
                        borderColor: this.tooltip.bordercolor,
                        borderWidth: this.tooltip.borderwidth,
                        style: {
                            fontSize: this.tooltip.style.fontsize,
                            color: this.tooltip.style.color,
                            fontWeight: this.tooltip.style.fontweight,
                        }
                    },
                    plotOptions: {
                        series: {
                            states: {
                                inactive: {
                                    enabled: false
                                }
                            },
                            // color: "blue",
                            dataLabels: {
                                enabled: this.plotOptions.series.dataLabels.enabled,
                                format: '{point.y}',
                                style: {
                                    fontSize: this.plotOptions.series.dataLabels.style.fontsize,
                                    color: this.plotOptions.series.dataLabels.style.color,
                                    textOutline: this.plotOptions.series.dataLabels.style.border + "px   "
                                        + this.plotOptions.series.dataLabels.style.bordercolor,
                                    fontWeight: this.plotOptions.series.dataLabels.style.fontweight,
                                }
                            },
                        },
                        area: {
                            depth: 0,
                        },
                        pie: {
                            slicedOffset: 0,
                            size: this.plotOptions.pie.size,
                            allowPointSelect: true,
                            cursor: 'pointer',
                            depth: this.plotOptions.pie.depth,
                            innerSize: this.plotOptions.pie.innersize,
                            dataLabels: {
                                format: '{point.' + this.plotOptions.pie.dataLabels.format + '}',
                                connectorWidth: this.plotOptions.pie.dataLabels.connectorwidth
                            }
                        }
                    },
                    xAxis: this.xAxis.map(e => {
                        return {
                            type: 'category',
                            crosshair: e.crosshair,
                            visible: e.enabled,
                            gridLineWidth: e.gridwidth,
                            alternateGridColor: e.gridcolor,
                            gridLineColor: e.gridlinecolor,
                            gridLineDashStyle: e.griddashtype,
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
                            labels: {
                                style: {
                                    fontSize: e.labels.style.fontsize,
                                    color: e.labels.style.color,
                                    textOutline: e.labels.style.border + "px   "
                                        + e.labels.style.bordercolor,
                                    fontWeight: e.labels.style.fontweight,
                                }
                            },

                        }
                    }),
                    yAxis: this.yAxis.map(e => {
                        return {
                            crosshair: e.crosshair,
                            tickAmount: e.tickcount,
                            gridLineWidth: e.gridwidth,
                            opposite: e.opposite, //在对面显示坐标轴 
                            visible: e.enabled,
                            min: e.min,
                            max: e.max,
                            alternateGridColor: e.gridcolor,
                            gridLineColor: e.gridlinecolor,
                            gridLineDashStyle: e.griddashtype,
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
                            labels: { //   format: '{value} mm',
                                style: {
                                    fontSize: e.labels.style.fontsize,
                                    color: e.labels.style.color,
                                    textOutline: e.labels.style.border + "px   "
                                        + e.labels.style.bordercolor,
                                    fontWeight: e.labels.style.fontweight,
                                }
                            },

                        }
                    }),
                    series: this.series.map(s => {
                        this.i++
                        return {
                            name: s.yname,
                            type: s.type,
                            xAxis: s.xAxis,
                            yAxis: s.yAxis,
                            color: s.color,
                            depth: s.depth,
                            dataLabels: {
                                enabled: s.enabled,
                                format: '{point.' + s.format + '}',
                            },
                            data: this.result.map(e => {
                                return { name: e[s.xname], y: e[s.yname] }
                            }),
                            events: {
                                click: function (event) {
                                    var params = s.eventparams == undefined ? "{}" : s.eventparams
                                    var temp = eval("(" + params + ")")
                                    switch (s.event) {
                                        case "showdetail":
                                            that.visible = true
                                            that.modaltitle = event.point.name
                                            that.params = params
                                            break;
                                        case "topage":
                                            window.open(that.$router.resolve({
                                                path: temp.path
                                            }).href, temp.type)
                                            break;
                                        default: break;
                                    }
                                }
                            },
                        }
                    })
                }
            },
        },
        methods: {
            getRefsAff() {
                return document.getElementById('fulls')
            },
            handleCancel() {
                this.visible = false;
            },

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
                        this.i++
                    })
                        .catch(error => {
                            console.log(error)
                            this.result = []
                            this.$message.error(this.ss.componentname + "NetWork Error")
                        })
                }
            },
        }
    }
</script>
