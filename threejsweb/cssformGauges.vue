<template>
    <div>
        <a-divider>Charts</a-divider>
        <el-collapse accordion>
            <el-collapse-item title="Credits">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="显示">
                        <a-switch default-checked @change="handle_change" v-model="credits.enabled" />
                    </a-form-model-item>
                    <a-form-model-item label="颜色">
                        <el-color-picker size="mini" v-model="credits.color" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="内容">
                        <a-input v-model="credits.text" @change="handle_change" />
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <div style="display: flex;justify-content: space-between;">
                            <a-slider style="width: 50%;" v-model="credits.fontsize" @change="handle_change" :min="10"
                                :max="60" />
                            <a-input-number style="width: 40%;" v-model="credits.fontsize" :min="10" :max="60"
                                @change="handle_change" />
                        </div>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <div style="display: flex;justify-content: space-between;">
                            <a-slider style="width: 50%;" v-model="credits.fontweight" @change="handle_change" :min="0"
                                :max="1000" />
                            <a-input-number style="width: 40%;" v-model="credits.fontweight" :min="0" :max="1000"
                                @change="handle_change" />
                        </div>
                    </a-form-model-item>
                    <a-form-model-item label="X偏移">
                        <div style="display: flex;justify-content: space-between;">
                            <a-slider style="width: 50%;" v-model="credits.positionx" @change="handle_change"
                                :min="-100" :max="100" />
                            <a-input-number style="width: 40%;" v-model="credits.positionx" :min="-100" :max="100"
                                @change="handle_change" />
                        </div>
                    </a-form-model-item>
                    <a-form-model-item label="Y偏移">
                        <div style="display: flex;justify-content: space-between;">
                            <a-slider style="width: 50%;" v-model="credits.positiony" @change="handle_change"
                                :min="-100" :max="100" />
                            <a-input-number style="width: 40%;" v-model="credits.positiony" :min="-100" :max="100"
                                @change="handle_change" />
                        </div>
                    </a-form-model-item>
                    <a-form-model-item label="Align">
                        <a-radio-group button-style="solid" v-model="credits.align" @change="handle_change">
                            <a-radio-button value="left">
                                LEFT
                            </a-radio-button>
                            <a-radio-button value="right">
                                RIGHT
                            </a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>
                    <a-form-model-item label="垂直Align">
                        <a-radio-group button-style="solid" v-model="credits.verticalalign" @change="handle_change">
                            <a-radio-button value="top">
                                TOP
                            </a-radio-button>
                            <a-radio-button value="bottom">
                                Bottom
                            </a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>
                </a-form-model>
            </el-collapse-item>
            <el-collapse-item title="Chart">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">

                    <a-form-model-item label="显示3D">
                        <a-switch default-checked @change="handle_change" v-model="chart.options3denabled" />
                    </a-form-model-item>
                    <a-form-model-item label="Alpha">
                        <el-slider :min="0" :max="45" @change="handle_change" v-model="chart.options3dalpha">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Beta">
                        <el-slider :min="-45" :max="45" @change="handle_change" v-model="chart.options3dbeta">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Depth">
                        <el-slider :min="0" :max="500" @change="handle_change" v-model="chart.options3ddepth">
                        </el-slider>
                    </a-form-model-item>
                    <div v-for="(item,index) in ['top','right','bottom','left']" :key="index+item">
                        <a-form-model-item :label="'Spacing'+item">
                            <a-slider v-model="chart['spacing'+item]" @change="handle_change" :min="-100" :max="100" />
                        </a-form-model-item>
                    </div>
                </a-form-model>
            </el-collapse-item>
            <el-collapse-item title="Tooltip">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="tooltip.bordercolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="边框粗细">
                        <el-slider :min="0" :max="40" v-model="tooltip.borderwidth" @change="handle_change">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="tooltip.style.color" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <el-slider :min="10" :max="60" v-model="tooltip.style.fontsize" @change="handle_change">
                        </el-slider>
                    </a-form-model-item>
                </a-form-model>
            </el-collapse-item>
        </el-collapse>


        <a-divider>plot Options</a-divider>
        <el-collapse accordion>
            <el-collapse-item title="通用数据列Series">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="文本顯示">
                        <a-switch default-checked @change="handle_change"
                            v-model="plotOptions.series.dataLabels.enabled" />
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <el-slider @change="handle_change" v-model="plotOptions.series.dataLabels.style.fontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="plotOptions.series.dataLabels.style.color"
                            @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <el-slider :min="0" :max="999" @change="handle_change"
                            v-model="plotOptions.series.dataLabels.style.fontweight">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体边框">
                        <el-slider :min="0" :max="30" @change="handle_change"
                            v-model="plotOptions.series.dataLabels.style.border">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="plotOptions.series.dataLabels.style.bordercolor"
                            @change="handle_change">
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="X偏移">
                        <el-slider v-model="plotOptions.series.dataLabels.x" @change="handle_change" :min="-100"
                            :max="100">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Y偏移">
                        <el-slider v-model="plotOptions.series.dataLabels.y" @change="handle_change" :min="-100"
                            :max="100">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Align">
                        <a-radio-group button-style="solid" v-model="plotOptions.series.dataLabels.align"
                            @change="handle_change">
                            <a-radio-button value="left"> LEFT </a-radio-button>
                            <a-radio-button value="right"> RIGHT </a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>
                    <a-form-model-item label="垂直Align">
                        <a-radio-group button-style="solid" v-model="plotOptions.series.dataLabels.verticalalign"
                            @change="handle_change">
                            <a-radio-button value="top"> TOP </a-radio-button>
                            <a-radio-button value="bottom"> Bottom </a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>

                </a-form-model>
            </el-collapse-item>
        </el-collapse>

        <a-divider>
            yAxis<el-button type="text" size="mini" @click="handle_addy">新增Y轴</el-button>
        </a-divider>
        <el-collapse accordion>
            <el-collapse-item :title="'yAxis'+(index+1)" v-for="(yitem,index) in yAxis" :key="index+'yitem'">
                <template slot="title">
                    <i v-if="yAxis.length>1" @click="handle_deletey(index)" class="el-icon-remove-outline"></i>
                    &nbsp;&nbsp;&nbsp;yAxis {{index+1}}
                </template>
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="坐标轴名">
                        <a-input @change="handle_change" v-model="yitem.titletext" />
                    </a-form-model-item>
                    <a-form-model-item label="对齐">
                        <el-radio-group @change="handle_change" v-model="yitem.titlealign" size="mini">
                            <el-radio-button label="low"></el-radio-button>
                            <el-radio-button label="middle"></el-radio-button>
                            <el-radio-button label="high"></el-radio-button>
                        </el-radio-group>
                    </a-form-model-item>
                    <a-form-model-item label="旋转">
                        <el-slider :min="0" :max="360" v-model="yitem.rotation" :step="10">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="offsetX">
                        <el-slider :min="-100" :max="100" v-model="yitem.offsetx">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="offsetY">
                        <el-slider :min="-100" :max="100" v-model="yitem.offsety">
                        </el-slider>
                    </a-form-model-item>

                    <a-form-model-item label="偏移">
                        <el-slider v-model="yitem.offset" @change="handle_change" :min="-100" :max="100">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="刻度总数">
                        <el-slider :min="0" :max="20" @change="handle_change" v-model="yitem.tickcount">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="刻度宽度">
                        <el-slider :min="0" :max="20" @change="handle_change" v-model="yitem.tickwidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="刻度颜色">
                        <el-color-picker size="mini" v-model="yitem.tickcolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="刻度长度">
                        <el-slider :min="0" :max="50" @change="handle_change" v-model="yitem.ticklength">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="刻度朝向">
                        <el-radio-group @change="handle_change" v-model="yitem.tickposition" size="mini">
                            <el-radio-button label="inside"></el-radio-button>
                            <el-radio-button label="outside"></el-radio-button>
                        </el-radio-group>
                    </a-form-model-item>


                    <a-form-model-item label="轴线宽">
                        <el-slider :min="0" :max="30" @change="handle_change" v-model="yitem.linewidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="轴线颜色">
                        <el-color-picker size="mini" v-model="yitem.linecolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="Min">
                        <a-input-number @change="handle_change" v-model="yitem.min" />
                    </a-form-model-item>
                    <a-form-model-item label="Max">
                        <a-input-number @change="handle_change" v-model="yitem.max" />
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <el-slider @change="handle_change" v-model="yitem.labels.style.fontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="yitem.labels.style.color" @change="handle_change"
                            show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <el-slider :min="0" :max="999" @change="handle_change" v-model="yitem.labels.style.fontweight">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体边框">
                        <el-slider :min="0" :max="30" @change="handle_change" v-model="yitem.labels.style.border">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="yitem.labels.style.bordercolor" @change="handle_change">
                        </el-color-picker>
                    </a-form-model-item>

                    <a-form-model-item label="Distance">
                        <el-slider v-model="yitem.distance" @change="handle_change" :min="-100" :max="100">
                        </el-slider>
                    </a-form-model-item>


                </a-form-model>
            </el-collapse-item>
        </el-collapse>


        <a-divider>
            Series <el-button type="text" size="mini" @click="handle_addseries">新增Series</el-button>
        </a-divider>
        <el-collapse accordion>
            <el-collapse-item :title="'Series'+(index+1)" v-for="(seriesitem,index) in series"
                :key="index+'seriesitem'">
                <template slot="title">
                    <i v-if="series.length>1" @click="handle_deleteseries(index)" class="el-icon-remove-outline"></i>
                    &nbsp;&nbsp;&nbsp;Series {{index+1}} -{{seriesitem.yname}}
                </template>
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="Type">
                        <el-select v-model="seriesitem.type" size="mini" placeholder="请选择">
                            <el-option v-for="item in seriestype" :key="item" :value="item">
                                <span style="float: left">{{ item }}</span>
                            </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="yname">
                        <a-input @change="handle_change" v-model="seriesitem.yname"></a-input>
                    </a-form-model-item>
                    <a-form-model-item label="y轴">
                        <el-select v-model="seriesitem.yAxis" size="mini" placeholder="请选择">
                            <el-option v-for="(m,i) in yAxis" :key="i" :value="i" :label="'yAxis'+(i+1)">
                                <span style="float: left"> yAxis {{i+1 }}</span>
                            </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="颜色">
                        <el-color-picker size="mini" v-model="seriesitem.color" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>

                    <a-form-model-item label="环">
                        <el-slider :max="150" @change="handle_change" v-model="seriesitem.radius">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="内环">
                        <el-slider :max="150" @change="handle_change" v-model="seriesitem.innerradius">
                        </el-slider>
                    </a-form-model-item>

                    <a-form-model-item label="X">
                        <el-slider :min="-100" :max="150" @change="handle_change" v-model="seriesitem.x">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Y">
                        <el-slider :max="150" @change="handle_change" v-model="seriesitem.y">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Align">
                        <a-radio-group button-style="solid" v-model="seriesitem.align" @change="handle_change">
                            <a-radio-button value="left"> LEFT </a-radio-button>
                            <a-radio-button value="right"> RIGHT </a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>
                    <a-form-model-item label="垂直Align">
                        <a-radio-group button-style="solid" v-model="seriesitem.verticalalign" @change="handle_change">
                            <a-radio-button value="top"> TOP </a-radio-button>
                            <a-radio-button value="bottom"> Bottom </a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <el-slider @change="handle_change" v-model="seriesitem.fontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="seriesitem.fontcolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <el-slider :min="0" :max="999" @change="handle_change" v-model="seriesitem.fontweight">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体边框">
                        <el-slider :min="0" :max="30" @change="handle_change" v-model="seriesitem.border">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="seriesitem.bordercolor" @change="handle_change">
                        </el-color-picker>
                    </a-form-model-item>
                    <el-divider content-position="left">指针</el-divider>
                    <a-form-model-item label="Radius">
                        <el-slider :max="200" :step="10" @change="handle_change" v-model="seriesitem.dialradius">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="背景颜色">
                        <el-color-picker size="mini" v-model="seriesitem.dialbackgroundcolor" @change="handle_change"
                            show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="seriesitem.dialbordercolor" @change="handle_change"
                            show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="边框宽度">
                        <el-slider :min="0" :max="10" @change="handle_change" v-model="seriesitem.dialborderwidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="底部宽度">
                        <el-slider :min="0" :max="10" @change="handle_change" v-model="seriesitem.dialbasewidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="顶部宽度">
                        <el-slider :min="0" :max="10" @change="handle_change" v-model="seriesitem.dialtopwidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="BaseLength">
                        <el-slider   @change="handle_change" v-model="seriesitem.diallbaselength">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="RearLength">
                        <el-slider   @change="handle_change" v-model="seriesitem.dialrearlength">
                        </el-slider>
                    </a-form-model-item>
                </a-form-model>
            </el-collapse-item>
        </el-collapse>



        <a-divider>Pane</a-divider>
        <el-collapse accordion>
            <el-collapse-item title="Pane">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="Center">
                        <div v-for="(item,index) in pane.center" :key="index">
                            <el-slider @change="handle_change" v-model="pane.center[index]">
                            </el-slider>
                        </div>
                    </a-form-model-item>
                    <a-form-model-item label="Size">
                        <el-slider @change="handle_change" v-model="pane.size">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="内环">
                        <el-slider @change="handle_change" v-model="pane.innerradius">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="外环">
                        <el-slider :max="150" @change="handle_change" v-model="pane.outerradius">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="背景颜色">
                        <el-color-picker size="mini" v-model="pane.backgroundcolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="边框粗细">
                        <el-slider :min="0" :max="50" @change="handle_change" v-model="pane.borderwidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="pane.bordercolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="形状">
                        <el-select v-model="pane.shape" size="mini" placeholder="请选择">
                            <el-option value="solid" label="圆形"> </el-option>
                            <el-option value="arc" label="坐标轴min to max"> </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="Start Angle">
                        <el-slider :min="-180" :max="180" :step="10" @change="handle_change" v-model="pane.startangle">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="End Angle">
                        <el-slider :min="-180" :max="180" :step="10" @change="handle_change" v-model="pane.endangle">
                        </el-slider>
                    </a-form-model-item>

                </a-form-model>
            </el-collapse-item>
        </el-collapse>



    </div>
</template>

<script>
    //import merge from 'deepmerge'

    export default {
        props: {
            prop_style: {
                type: Object, default: () => ({})
            },
            keyid: { type: Number, default: 0, },
        },

        data() {
            return {
                color: '#e2e2e2',
                activeNames: ['1'], 
                seriestype: [
                    'gauge', 'solidgauge'
                ],
            };
        },
        created() {
            //this.$emit("highchart", { style: this.total_style, key: this.keyid })
            // console.log(this.prop_style)

        },
        computed: {
            credits: {
                get: function () {
                    return this.prop_style.credits
                },
                set: function () { }
            },
            chart: {
                get: function () {
                    return this.prop_style.chart
                },
                set: function () { }
            },
            tooltip: {
                get: function () {
                    return this.prop_style.tooltip
                },
                set: function () { }
            },
            plotOptions: {
                get: function () {
                    return this.prop_style.plotOptions
                },
                set: function () { }
            },
            xAxis: {
                get: function () {
                    return this.prop_style.xAxis
                },
                set: function () { }
            },
            yAxis: {
                get: function () {
                    return this.prop_style.yAxis
                },
                set: function () { }
            },
            series: {
                get: function () {
                    return this.prop_style.series
                },
                set: function () { }
            },
            legend: {
                get: function () {
                    return this.prop_style.legend
                },
                set: function () { }
            },
            pane: {
                get: function () {
                    return this.prop_style.pane
                },
                set: function () { }
            },
            total_style() {
                return {
                    credits: this.credits,
                    chart: this.chart,
                    tooltip: this.tooltip,
                    plotOptions: this.plotOptions,
                    xAxis: this.xAxis,
                    yAxis: this.yAxis,
                    series: this.series,
                    legend: this.legend,
                    pane: this.pane
                }
            },
        },
        methods: {
            handleChange(val) {
                console.log(val);
            },
            handle_change() {
                this.$emit("highchart", { style: this.total_style, key: this.keyid })
            },
            handlePushKeyword() {
                if (event.keyCode === 13) {
                    event.preventDefault();
                    return false;
                }
            },
            handle_addx() {
                var temp = JSON.parse(JSON.stringify(this.xAxis[0]))
                this.xAxis.push(temp)
            },
            handle_deletex(e) {
                this.xAxis.splice(e, 1)
            },
            handle_addy() {
                var temp = JSON.parse(JSON.stringify(this.yAxis[0]))
                this.yAxis.push(temp)
            },
            handle_deletey(e) {
                this.yAxis.splice(e, 1)
            },
            handle_addseries() {
                var temp = JSON.parse(JSON.stringify(this.series[0]))
                this.series.push(temp)
            },
            handle_deleteseries(e) {
                this.series.splice(e, 1)
            },
        },
    };
</script>

<style lang="less" scoped>
    .paneltop {
        background-color: @primary-6;

    }
</style>
