<template>
    <div>
        <el-collapse accordion>
            <el-collapse-item title="Row Cell">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="文字颜色">
                        <el-color-picker size="mini" v-model="rowcell.color" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="文字大小">
                        <el-slider @change="handle_change" v-model="rowcell.fontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="背景颜色">
                        <el-color-picker size="mini" v-model="rowcell.backgroundcolor" @change="handle_change"
                            show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="行高">
                        <el-slider @change="handle_change" v-model="rowcell.height">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <el-slider default-value="12" :max="999" @change="handle_change" v-model="rowcell.fontweight">
                        </el-slider>
                    </a-form-model-item>
                </a-form-model>
            </el-collapse-item>
            <el-collapse-item title="Header Cell">
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="文字颜色">
                        <el-color-picker size="mini" v-model="headercell.color" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="文字大小">
                        <el-slider @change="handle_change" v-model="headercell.fontsize">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="背景颜色">
                        <el-color-picker size="mini" v-model="headercell.backgroundcolor" @change="handle_change"
                            show-alpha>
                        </el-color-picker>
                    </a-form-model-item>
                    <a-form-model-item label="行高">
                        <el-slider @change="handle_change" v-model="headercell.height">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字体粗细">
                        <el-slider default-value="12" :max="999" @change="handle_change" v-model="headercell.fontweight">
                        </el-slider>
                    </a-form-model-item>
                </a-form-model>
            </el-collapse-item>
        </el-collapse>


        <a-divider>
            Column <el-button type="text" size="mini" @click="handle_addcolumn">新增Series</el-button>
        </a-divider>
        <el-collapse accordion>
            <el-collapse-item :title="'Column'+(index+1)" v-for="(item,index) in columns" :key="index+'item'">
                <template slot="title">
                    <i v-if="columns.length>1" @click="handle_deletecolumn(index)" class="el-icon-remove-outline"></i>
                    &nbsp;&nbsp;&nbsp;Column {{index+1}} -{{item.label}}
                </template>
                <a-form-model layout="horizontal" :label-col=" { span: 8 }" :wrapper-col=" { span: 16 }">
                    <a-form-model-item label="列名">
                        <a-input @change="handle_change" v-model="item.label"></a-input>
                    </a-form-model-item>
                    <a-form-model-item label="列宽">
                        <el-slider :min="10" :max="300" @change="handle_change" v-model="item.width">
                        </el-slider>
                    </a-form-model-item>
                    <a-form-model-item label="字段名">
                        <a-input @change="handle_change" v-model="item.prop"></a-input>
                    </a-form-model-item>
                    <a-form-model-item label="对齐">
                        <a-radio-group button-style="solid" v-model="item.align" @change="handle_change">
                            <a-radio-button value="left">
                                <a-icon type="align-left" />
                            </a-radio-button>
                            <a-radio-button value="center">
                                <a-icon type="align-center" />
                            </a-radio-button>
                            <a-radio-button value="right">
                                <a-icon type="align-right" />
                            </a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>
                    <a-form-model-item label="字体颜色">
                        <el-color-picker size="mini" v-model="item.color" @change="handle_change" show-alpha>
                        </el-color-picker>
                    </a-form-model-item>

                    <a-form-model-item label="字体粗细">
                        <el-slider default-value="12" :max="999" @change="handle_change" v-model="item.fontweight">
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
                coltype: [
                    'tag'
                ],
            };
        },
        created() {

        },
        computed: {
            columns: {
                get: function () {
                    return this.prop_style.columns
                },
                set: function () { }
            },
            rowcell: {
                get: function () {
                    return this.prop_style.rowcell
                },
                set: function () { }
            },
            headercell: {
                get: function () {
                    return this.prop_style.headercell
                },
                set: function () { }
            },
            total_style() {
                return {
                    columns: this.columns,
                    rowcell: this.rowcell,
                    headercell: this.headercell,
                }
            },
        },
        methods: {
            handle_change() {
                this.$emit("eltable", { style: this.total_style, key: this.keyid })
            },
            handle_addcolumn() {
                var temp = JSON.parse(JSON.stringify(this.columns[0]))
                this.columns.push(temp)
            },
            handle_deletecolumn(e) {
                this.columns.splice(e, 1)
            },
        },
    };
</script>

<style lang="less" scoped>
    .paneltop {
        background-color: @primary-6;

    }
</style>
