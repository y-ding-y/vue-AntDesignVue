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
                    <div v-for="(item,index) in ['bottom','side','back']" :key="index+item">
                        <a-form-model-item :label="item">
                            <div style="display: flex;justify-content: space-between;">
                                <el-color-picker size="mini" v-model="chart.frame[item].color" @change="handle_change"
                                    show-alpha></el-color-picker>
                                <a-slider style="width: 70%;" v-model="chart.frame[item].size" @change="handle_change"
                                    :min="0" :max="30" />
                                <a-switch default-checked @change="handle_change" v-model="chart.frame[item].enabled" />
                            </div>
                        </a-form-model-item>
                    </div>
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
            <el-collapse-item title="Legend">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="图例">
                        <a-switch default-checked @change="handle_change" v-model="legend.enabled" />
                    </a-form-model-item>
                    <a-form-model-item label="悬浮">
                        <a-switch default-checked @change="handle_change" v-model="legend.floating" />
                    </a-form-model-item>
                    <a-form-model-item label="X偏移">
                        <el-slider :min="-100" :max="100" @change="handle_change" v-model="legend.x">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Y偏移">
                        <el-slider :min="-100" :max="100" @change="handle_change" v-model="legend.y">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="legend.color" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="悬停颜色">
                        <el-color-picker size="mini" v-model="legend.hovercolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <el-slider @change="handle_change" v-model="legend.fontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Align">
                        <a-radio-group button-style="solid" v-model="legend.align" @change="handle_change">
                            <a-radio-button value="left"> LEFT </a-radio-button>
                            <a-radio-button value="right"> RIGHT </a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>
                    <a-form-model-item label="垂直Align">
                        <a-radio-group button-style="solid" v-model="legend.verticalalign" @change="handle_change">
                            <a-radio-button value="top"> TOP </a-radio-button>
                            <a-radio-button value="bottom"> Bottom </a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>
                    <a-form-model-item label="Layout">
                        <a-radio-group button-style="solid" v-model="legend.layout" @change="handle_change">
                            <a-radio-button value="horizontal"> 水平 </a-radio-button>
                            <a-radio-button value="vertical"> 垂直 </a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>

                    <!-- <a-form-model-item label="字体粗细">
                        <el-slider :min="0" :max="999" @change="handle_change" v-model="legend.labels.style.fontweight">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体边框">
                        <el-slider :min="0" :max="30" @change="handle_change" v-model="legend.labels.style.border">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="legend.labels.style.bordercolor" @change="handle_change">
                        </el-color-picker>
                    </a-form-model-item> -->
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
                        {{plotOptions.series.dataLabels.style.bordercolor}}
                        <el-color-picker size="mini" v-model="plotOptions.series.dataLabels.style.bordercolor"
                            @change="handle_change">
                        </el-color-picker>
                    </a-form-model-item>
                </a-form-model>
            </el-collapse-item>
            <el-collapse-item title="饼图Pie">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="Size">
                        <el-slider :min="0" :max="1000" @change="handle_change" v-model="plotOptions.pie.size">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Depth">
                        <el-slider @change="handle_change" v-model="plotOptions.pie.depth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="內圈">
                        <el-slider :min="0" :max="500" @change="handle_change" v-model="plotOptions.pie.innersize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="指示線粗細">
                        <el-slider :min="0" :max="30" @change="handle_change"
                            v-model="plotOptions.pie.dataLabels.connectorwidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Format">
                        <a-input @keydown="handlePushKeyword($event)" @keyup.enter.native="handle_change"
                            v-model="plotOptions.pie.dataLabels.format"></a-input>
                    </a-form-model-item>
                </a-form-model>
            </el-collapse-item>
            <el-collapse-item title="面积图Area">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="Size">
                        <el-slider :min="0" :max="1000" @change="handle_change" v-model="plotOptions.pie.size">
                        </el-slider>
                    </a-form-model-item>
                    <!-- <a-form-model-item label="Depth">
                        <el-slider @change="handle_change" v-model="plotOptions.area.depth">
                        </el-slider>
                    </a-form-model-item> -->
                    <a-form-model-item label="內圈">
                        <el-slider :min="0" :max="500" @change="handle_change" v-model="plotOptions.pie.innersize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="指示線粗細">
                        <el-slider :min="0" :max="30" @change="handle_change"
                            v-model="plotOptions.pie.dataLabels.connectorwidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Format">
                        <a-input @keydown="handlePushKeyword($event)" @keyup.enter.native="handle_change"
                            v-model="plotOptions.pie.dataLabels.format"></a-input>
                    </a-form-model-item>
                </a-form-model>
            </el-collapse-item>
        </el-collapse>


        <a-divider>
            xAxis <el-button type="text" size="mini" @click="handle_addx">新增X轴</el-button>
        </a-divider>
        <el-collapse accordion>
            <el-collapse-item :title="'xAxis'+(index+1)" v-for="(xitem,index) in xAxis" :key="index+'xitem'">
                <template slot="title">
                    <i v-if="xAxis.length>1" @click="handle_deletex(index)" class="el-icon-remove-outline"></i>
                    &nbsp;&nbsp;&nbsp;xAxis {{index+1}}
                </template>
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="显示坐标">
                        <a-switch default-checked @change="handle_change" v-model="xitem.enabled" />
                    </a-form-model-item>
                    <a-form-model-item label="Crosshair">
                        <a-switch default-checked @change="handle_change" v-model="xitem.crosshair" />
                    </a-form-model-item>
                    <a-form-model-item label="坐标轴名">
                        <a-input @change="handle_change" v-model="xitem.titletext" />
                    </a-form-model-item>
                    <a-form-model-item label="对齐">
                        <el-radio-group @change="handle_change" v-model="xitem.titlealign" size="mini">
                            <el-radio-button label="low"></el-radio-button>
                            <el-radio-button label="middle"></el-radio-button>
                            <el-radio-button label="high"></el-radio-button>
                        </el-radio-group>
                    </a-form-model-item>
                    <a-form-model-item label="旋转">
                        <el-slider :min="0" :max="360" v-model="xitem.rotation" :step="10">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="offsetX">
                        <el-slider :min="-100" :max="100" v-model="xitem.offsetx">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="offsetY">
                        <el-slider :min="-100" :max="100" v-model="xitem.offsety">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Opposite" v-if="yAxis.length>1">
                        <a-switch :default-checked="false" @change="handle_change" v-model="xitem.opposite" />
                    </a-form-model-item>
                    <a-form-model-item label="间隔颜色">
                        <el-color-picker size="mini" v-model="xitem.gridcolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="刻度线宽">
                        <el-slider :min="0" :max="30" @change="handle_change" v-model="xitem.gridwidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="刻度颜色">
                        <el-color-picker size="mini" v-model="xitem.gridlinecolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="刻度样式">
                        <el-select v-model="xitem.griddashtype" size="mini" placeholder="请选择">
                            <el-option v-for="item in dashStyles" :key="item" :value="item">
                                <span style="float: left">{{ item }}</span>
                            </el-option>
                        </el-select>
                    </a-form-model-item>
                    <a-form-model-item label="字体大小">
                        <el-slider @change="handle_change" v-model="xitem.labels.style.fontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="xitem.labels.style.color" @change="handle_change"
                            show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <el-slider :min="0" :max="999" @change="handle_change" v-model="xitem.labels.style.fontweight">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体边框">
                        <el-slider :min="0" :max="30" @change="handle_change" v-model="xitem.labels.style.border">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="边框颜色">
                        <el-color-picker size="mini" v-model="xitem.labels.style.bordercolor" @change="handle_change">
                        </el-color-picker>
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
                    <a-form-model-item label="显示坐标">
                        <a-switch default-checked @change="handle_change" v-model="yitem.enabled" />
                    </a-form-model-item>
                    <a-form-model-item label="Crosshair">
                        <a-switch default-checked @change="handle_change" v-model="yitem.crosshair" />
                    </a-form-model-item>
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
                    <a-form-model-item label="Opposite" v-if="yAxis.length>1">
                        <a-switch :default-checked="false" @change="handle_change" v-model="yitem.opposite" />
                    </a-form-model-item>
                    <a-form-model-item label="刻度线数">
                        <el-slider :min="0" :max="10" @change="handle_change" v-model="yitem.tickcount">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="Min">
                        <a-input-number @change="handle_change" v-model="yitem.min" />
                    </a-form-model-item>
                    <a-form-model-item label="Max">
                        <a-input-number @change="handle_change" v-model="yitem.max" />
                    </a-form-model-item>
                    <a-form-model-item label="间隔颜色">
                        <el-color-picker size="mini" v-model="yitem.gridcolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="刻度线宽">
                        <el-slider :min="0" :max="30" @change="handle_change" v-model="yitem.gridwidth">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="刻度颜色">
                        <el-color-picker size="mini" v-model="yitem.gridlinecolor" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="刻度样式">
                        <el-select v-model="yitem.griddashtype" size="mini" placeholder="请选择">
                            <el-option v-for="item in dashStyles" :key="item" :value="item">
                                <span style="float: left">{{ item }}</span>
                            </el-option>
                        </el-select>
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

                    <a-form-model-item label="显示文本">
                        <a-switch default-checked @change="handle_change" v-model="seriesitem.enabled" />
                    </a-form-model-item>
                    <a-form-model-item label="文本内容">
                        <a-input @change="handle_change" v-model="seriesitem.format"></a-input>
                    </a-form-model-item>
                    <a-form-model-item label="xname">
                        <a-input @change="handle_change" v-model="seriesitem.xname"></a-input>
                    </a-form-model-item>
                    <a-form-model-item label="yname">
                        <a-input @change="handle_change" v-model="seriesitem.yname"></a-input>
                    </a-form-model-item>
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
                    <a-form-model-item label="Depth">
                        <el-slider v-model="seriesitem.depth">
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
                dashStyles: [
                    'Solid',
                    'ShortDash',
                    'ShortDot',
                    'ShortDashDot',
                    'ShortDashDotDot',
                    'Dot',
                    'Dash',
                    'LongDash',
                    'DashDot',
                    'LongDashDot',
                    'LongDashDotDot'
                ],
                seriestype: [
                    'bar', 'column', 'line', 'pie', 'area'
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
