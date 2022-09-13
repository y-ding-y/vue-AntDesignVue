<template>
    <div style="margin:-16px">
        <a-divider>Charts</a-divider>
        <a-collapse accordion>
            <a-collapse-panel header="Chart">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="height">
                        <a-slider v-model="chart.height" @change="handle_change" />
                    </a-form-model-item>
                    <a-form-model-item label="width">
                        <a-slider v-model="chart.width" @change="handle_change" />
                    </a-form-model-item>
                    <div v-for="(item,index) in ['top','left']" :key="index+item">
                        <a-form-model-item :label="item">
                            <a-slider v-model="chart['spacing'+item]" @change="handle_change" />
                        </a-form-model-item>
                    </div>
                </a-form-model>
            </a-collapse-panel>
            <a-collapse-panel header="Tooltip">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="Trigger">
                        <el-select v-model="tooltip.trigger" size="mini" placeholder="请选择" @change="handle_change">
                            <el-option v-for="item in triggerlist" :key="item" :value="item">
                                <span style="float: left">{{ item }}</span>
                            </el-option>
                        </el-select>
                    </a-form-model-item>
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
            </a-collapse-panel>
            <a-collapse-panel header="Legend">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="图例">
                        <a-switch default-checked @change="handle_change" v-model="legend.enabled" />
                    </a-form-model-item>
                    <a-form-model-item label="Left">
                        <el-slider @input="handle_change" v-model="legend.x">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Top">
                        <el-slider @input="handle_change" v-model="legend.y">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="legend.color" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <el-slider @input="handle_change" v-model="legend.fontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Layout">
                        <a-radio-group button-style="solid" v-model="legend.layout" @change="handle_change">
                            <a-radio-button value="horizontal"> 水平 </a-radio-button>
                            <a-radio-button value="vertical"> 垂直 </a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>
                </a-form-model>
            </a-collapse-panel>
        </a-collapse>

        <a-divider>
            xAxis <el-button type="text" size="mini" @click="handle_addx">新增X轴</el-button>
        </a-divider>
        <a-collapse accordion>
            <a-collapse-panel :header="'xAxis'+(index+1)" v-for="(xitem,index) in xAxis" :key="index+'xitem'">
                <template slot="title">
                    <i v-if="xAxis.length>1" @click="handle_deletex(index)" class="el-icon-remove-outline"></i>
                    &nbsp;&nbsp;&nbsp;xAxis {{index+1}}
                </template>
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="显示坐标">
                        <a-switch default-checked @change="handle_change" v-model="xitem.enabled" />
                    </a-form-model-item>
                    <a-form-model-item label="Tick">
                        <el-slider @input="handle_change" :max="50" v-model="xitem.tickcount">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="坐标轴名">
                        <a-input @change="handle_change" v-model="xitem.titletext" />
                    </a-form-model-item>
                    <a-form-model-item label="对齐">
                        <el-radio-group @change="handle_change" v-model="xitem.titlealign" size="mini">
                            <el-radio-button label="start"></el-radio-button>
                            <el-radio-button label="center"></el-radio-button>
                            <el-radio-button label="end"></el-radio-button>
                        </el-radio-group>
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <el-slider @input="handle_change" v-model="xitem.namefontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="xitem.namecolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <el-slider :min="0" :max="999" @input="handle_change" v-model="xitem.namefontweight">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体边框">
                        <el-slider :min="0" :max="30" @input="handle_change" v-model="xitem.nameborderwidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="xitem.namebordercolor" @change="handle_change">
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="旋转">
                        <el-slider :min="0" :max="360" v-model="xitem.rotation" :step="10" @input="handle_change">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="显示">
                        <a-switch default-checked @change="handle_change" v-model="xitem.splitline.show" />
                    </a-form-model-item>
                    <a-form-model-item label="类型">
                        <el-select v-model="xitem.splitline.type" size="mini" placeholder="请选择" @change="handle_change">
                            <el-option v-for="item in dashStyles" :key="item" :value="item">
                                <span style="float: left">{{ item }}</span>
                            </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="颜色">
                        <el-color-picker size="mini" v-model="xitem.splitline.color" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="粗细">
                        <el-slider :min="0" :max="10" @input="handle_change" v-model="xitem.splitline.width">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <el-slider @input="handle_change" v-model="xitem.labels.style.fontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="xitem.labels.style.color" @change="handle_change"
                            show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <el-slider :min="0" :max="999" @input="handle_change" v-model="xitem.labels.style.fontweight">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体边框">
                        <el-slider :min="0" :max="30" @input="handle_change" v-model="xitem.labels.style.border">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="xitem.labels.style.bordercolor" @change="handle_change">
                        </el-color-picker>
                    </a-form-model-item>
                </a-form-model>
            </a-collapse-panel>
        </a-collapse>


        <a-divider>
            yAxis<el-button type="text" size="mini" @click="handle_addy">新增Y轴</el-button>
        </a-divider>
        <a-collapse accordion>
            <a-collapse-panel :header="'yAxis'+(index+1)" v-for="(yitem,index) in yAxis" :key="index+'yitem'">
                <template slot="title">
                    <i v-if="yAxis.length>1" @click="handle_deletey(index)" class="el-icon-remove-outline"></i>
                    &nbsp;&nbsp;&nbsp;yAxis {{index+1}}
                </template>
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="显示坐标">
                        <a-switch default-checked @change="handle_change" v-model="yitem.enabled" />
                    </a-form-model-item>
                    <a-form-model-item label="Tick">
                        <el-slider @input="handle_change" :max="50" v-model="yitem.tickcount">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="坐标轴名">
                        <a-input @change="handle_change" v-model="yitem.titletext" />
                    </a-form-model-item>
                    <a-form-model-item label="对齐">
                        <el-radio-group @change="handle_change" v-model="yitem.titlealign" size="mini">
                            <el-radio-button label="start"></el-radio-button>
                            <el-radio-button label="center"></el-radio-button>
                            <el-radio-button label="end"></el-radio-button>
                        </el-radio-group>
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <el-slider @input="handle_change" v-model="yitem.namefontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="yitem.namecolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <el-slider :min="0" :max="999" @input="handle_change" v-model="yitem.namefontweight">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体边框">
                        <el-slider :min="0" :max="30" @input="handle_change" v-model="yitem.nameborderwidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="yitem.namebordercolor" @change="handle_change">
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="旋转">
                        <el-slider :min="0" :max="360" v-model="yitem.rotation" :step="10" @input="handle_change">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Min">
                        <a-input-number @change="handle_change" v-model="yitem.min" />
                    </a-form-model-item>
                    <a-form-model-item label="Max">
                        <a-input-number @change="handle_change" v-model="yitem.max" />
                    </a-form-model-item>

                    <a-form-model-item label="显示">
                        <a-switch default-checked @change="handle_change" v-model="yitem.splitline.show" />
                    </a-form-model-item>
                    <a-form-model-item label="类型">
                        <el-select v-model="yitem.splitline.type" size="mini" placeholder="请选择" @change="handle_change">
                            <el-option v-for="item in dashStyles" :key="item" :value="item">
                                <span style="float: left">{{ item }}</span>
                            </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="颜色">
                        <el-color-picker size="mini" v-model="yitem.splitline.color" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="粗细">
                        <el-slider :min="0" :max="10" @input="handle_change" v-model="yitem.splitline.width">
                        </el-slider>
                    </a-form-model-item>

                    <a-form-model-item label="字体大小">
                        <el-slider @input="handle_change" v-model="yitem.labels.style.fontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="yitem.labels.style.color" @change="handle_change"
                            show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <el-slider :min="0" :max="999" @input="handle_change" v-model="yitem.labels.style.fontweight">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体边框">
                        <el-slider :min="0" :max="30" @input="handle_change" v-model="yitem.labels.style.border">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="yitem.labels.style.bordercolor" @change="handle_change">
                        </el-color-picker>
                    </a-form-model-item>
                </a-form-model>
            </a-collapse-panel>
        </a-collapse>

        <a-divider>
            Series <el-button type="text" size="mini" @click="handle_addseries">新增Series</el-button>
        </a-divider>
        <a-collapse accordion>
            <a-collapse-panel :header="'Series'+(index+1+'-'+seriesitem.yname)" v-for="(seriesitem,index) in series"
                :key="index+'seriesitem'">
                <i slot="extra" v-if="series.length>1" @click="handle_deleteseries(index)"
                    class="el-icon-remove-outline"></i>
                &nbsp;&nbsp;&nbsp;
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="Type">
                        <el-select v-model="seriesitem.type" size="mini" placeholder="请选择" @change="handle_change">
                            <el-option v-for="item in seriestype" :key="item" :value="item">
                                <span style="float: left">{{ item }}</span>
                            </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="xname">
                        <a-input @change="handle_change" v-model="seriesitem.xname"></a-input>
                    </a-form-model-item>
                    <a-form-model-item label="yname">
                        <a-input @change="handle_change" v-model="seriesitem.yname"></a-input>
                    </a-form-model-item>
                    <div v-if="seriesitem.type=='pie'">
                        <a-form-model-item label="内半径">
                            <el-slider @input="handle_change" v-model="seriesitem.radius1">
                            </el-slider>
                        </a-form-model-item>
                        <a-form-model-item label="外半径">
                            <el-slider @input="handle_change" v-model="seriesitem.radius2">
                            </el-slider>
                        </a-form-model-item>
                        <a-form-model-item label="center1">
                            <el-slider @input="handle_change" v-model="seriesitem.center1">
                            </el-slider>
                        </a-form-model-item>
                        <a-form-model-item label="center2">
                            <el-slider @input="handle_change" v-model="seriesitem.center2">
                            </el-slider>
                        </a-form-model-item>
                    </div>
                    <div v-else>
                        <a-form-model-item label="堆叠">
                            <a-input @change="handle_change" v-model="seriesitem.stack"></a-input>
                        </a-form-model-item>
                    </div>

                    <a-form-model-item label="x轴">
                        <el-select v-model="seriesitem.xAxis" size="mini" placeholder="请选择">
                            <el-option v-for="(m,i) in xAxis" :key="i" :value="i" :label="'xAxis'+(i+1)">
                                <span style="float: left"> xAxis {{i+1 }}</span>
                            </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="y轴">
                        <el-select v-model="seriesitem.yAxis" size="mini" placeholder="请选择">
                            <el-option v-for="(m,i) in yAxis" :key="i" :value="i" :label="'yAxis'+(i+1)">
                                <span style="float: left"> yAxis {{i+1 }}</span>
                            </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="series颜色">
                        <el-color-picker size="mini" v-model="seriesitem.color" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="阶梯点">
                        <el-select v-model="seriesitem.step" size="mini" placeholder="请选择" @change="handle_change">
                            <el-option value="false"> </el-option>
                            <el-option value="start"> </el-option>
                            <el-option value="middle"> </el-option>
                            <el-option value="end"> </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="显示文本">
                        <a-switch default-checked @change="handle_change" v-model="seriesitem.enabled" />
                    </a-form-model-item>
                    <a-form-model-item label="文本内容">
                        <a-input @change="handle_change" v-model="seriesitem.format"></a-input>
                    </a-form-model-item>
                    <a-form-model-item label="Position">
                        <el-select v-model="seriesitem.position" size="mini" placeholder="请选择" @change="handle_change">
                            <el-option v-for="item in labelposition" :key="item" :value="item">
                                <span style="float: left">{{ item }}</span>
                            </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <el-slider @input="handle_change" v-model="seriesitem.labelfontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="seriesitem.labelcolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <el-slider :min="0" :max="999" @input="handle_change" v-model="seriesitem.labelfontweight">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体边框">
                        <el-slider :min="0" :max="30" @input="handle_change" v-model="seriesitem.labelborder">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="seriesitem.labelbordercolor" @change="handle_change">
                        </el-color-picker>
                    </a-form-model-item>

                    <a-form-model-item label="显示引导线">
                        <a-switch default-checked @change="handle_change" v-model="seriesitem.lablelineshow" />
                    </a-form-model-item>
                    <a-form-model-item label="类型">
                        <el-select v-model="seriesitem.lablelinetype" size="mini" placeholder="请选择"
                            @change="handle_change">
                            <el-option v-for="item in dashStyles" :key="item" :value="item">
                                <span style="float: left">{{ item }}</span>
                            </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="颜色">
                        <el-color-picker size="mini" v-model="seriesitem.lablelinecolor" @change="handle_change"
                            show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="Length">
                        <el-slider :min="0" :max="50" @input="handle_change" v-model="seriesitem.lablelinelength">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Length2">
                        <el-slider :min="0" :max="50" @input="handle_change" v-model="seriesitem.lablelinelength2">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="粗细">
                        <el-slider :min="0" :max="10" @input="handle_change" v-model="seriesitem.lablelinewidth">
                        </el-slider>
                    </a-form-model-item>

                    <a-form-model-item label="Events">
                        <el-select v-model="seriesitem.event" size="mini" placeholder="请选择">
                            <el-option v-for="item in seriesevent" :key="item" :value="item">
                                <span style="float: left">{{ item }}</span>
                            </el-option>
                        </el-select>

                        <a-input type="textarea" @keydown="handlePushKeyword($event)"
                            @keyup.enter.native="handle_change" v-if="seriesitem.event=='showdetail'"
                            v-model="seriesitem.eventparams">
                        </a-input>
                        <a-input type="textarea" @keydown="handlePushKeyword($event)"
                            @keyup.enter.native="handle_change" v-if="seriesitem.event=='topage'"
                            v-model="seriesitem.eventparams">
                        </a-input>
                    </a-form-model-item>

                </a-form-model>
            </a-collapse-panel>
        </a-collapse>


    </div>
</template>

<script>
    //import merge from 'deepmerge'

    export default {
        name: "cssformecharts",
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
                triggerlist: ['none', 'axis', 'item'],
                dashStyles: ['solid', 'dotted', 'dashed',],
                seriestype: [
                    'bar', 'line', 'pie', 'area'
                ],
                seriesevent: ['showdetail', 'topage'],
                labelposition: ['top', 'left', 'right', 'bottom',
                    'inside', 'insideLeft', 'insideRight', 'insideTop', 'insideBottom',
                    'insideTopLeft', 'insideBottomLeft', 'insideTopRight', 'insideBottomRight', 'outside', 'center'],
            };
        },
        created() {
        },
        computed: {
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
                    chart: this.chart,
                    tooltip: this.tooltip,
                    xAxis: this.xAxis,
                    yAxis: this.yAxis,
                    series: this.series,
                    legend: this.legend,
                    pane: this.pane
                }
            },
        },
        methods: {
            formatTooltip(val) {
                return val / 10;
            },
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
