<template>
    <ef-main :id="imageId" :isMainPage="true" :title="$t('charttitle')" :loading="loading" :openSubmit="true"
        :form-items="formItems" :dataTableId="tableId" :imageDomId="imageId" @submit="submit" @resize="resize">
        <!-- <v-pie :type="'model'" ></v-pie>
            <v-pie :type="'cause'" ></v-pie> -->

        <ef-chart class="basebg" :height="chartHeight" :padding="[24, 64, 48, 64]" :series="[
        {
          quickType: 'bar',
          position: 'name*qty',
          color: 'category',
          adjust: [
            {
              type: 'dodge',
              marginRatio: 1 / 32,
            },
          ],
        },
      ]" :data="chartData">
        </ef-chart>
        <a-button @click="handle_detail()">詳情</a-button> 
        <br>
        <br>
        <a-row>
            <a-col :span="8">
                <v-pie v-if="i > 0" :type="'customer'" :category="category" :title="'客户分布图'" :key="i" :paras="paras">
                </v-pie> 
            </a-col>
            <a-col :span="8">
                <v-pie v-if="i > 0" :type="'model'" :category="category" :title="'机种分布图'" :key="i" :paras="paras">
                </v-pie> 
            </a-col>
            <a-col :span="8">
                <v-pie v-if="i > 0" :type="'cause'" :category="category" :title="'原因分布图'" :key="i" :paras="paras">
                </v-pie>
            </a-col>
        </a-row>

        <!-- <v-detail v-if="visible"  :visible="visible"  :title="'Detail'"
        :action-fix="'oqadetail'" :paras="paras"></v-detail> -->
    </ef-main>
</template>

<script>
    import { reqPost } from "@/services/base";
    import EfMain from "@/components/page/main";
    import Chart from "@/components/chart/Chart";
    import { merge } from "@/utils/util";
    import VPie from "./oqapie.vue";
    //import VDetail from "./oqadetail.vue";

    export default {
        name: "oqachart",
        i18n: merge(require("@/i18n"), require("./i18n")),
        components: { EfMain, EfChart: Chart, VPie,  },//VDetail
        data() {
            return {
                imageId: "fpyimage" + new Date().format("hiu"),
                tableId: "fpytable" + new Date().format("hiu"),
                loadcount: 0,
                sSize: "middle",

                rows: [],
                sortOrder: "",
                footerRowKey: "",
                chartData: [],
                key: 0,
                paras: {},
                values: {},
                rcs: [],
                category: "",
                i: 0,

                visible:false,
            };
        },
        computed: {
            title() {
                return this.$t("defectTitle");
            },
            loading() {
                return this.loadcount > 0;
            },
            chartHeight() {
                return this.sSize === "default"
                    ? 400
                    : this.sSize === "middle"
                        ? 300
                        : 200;
            },
            formItems() {
                var year = new Date().format("Y");
                return [
                    {
                        key: "date",
                        dataIndex: "workTime",
                        type: "range",
                        format: "YYYY-MM-DD",
                        maxDays: 366,
                        value: [new Date(year, 0, 1), new Date(year, 11, 31)],
                        allowClear: false,
                    },
                    {
                        key: "category",
                        type: "select",
                        paraKey: "para.123",
                        value: [],
                        mode: "multiple",
                        allowClear: false,
                    },
                ];
            },
        },
        watch: {},
        created() { },
        mounted() { },
        methods: {
            submit(values) {
                this.paras = { ...values };
                this.category = values.category;
                this.i++;
                this.handle_data(values);
            },
            resize(size) {
                this.sSize = size;
            },

            handle_data(values) {
                values.prop = "chart";
                this.loadcount = 1;
                reqPost("FILE_ACTION", values).then((res) => {
                    this.loadcount--;
                    this.chartData = res.data.map((e) => {
                        e.qty = Number(e.qty);
                        return e;
                    });
                });
            },

            handle_detail( ) {
                window.open("../../../quality/oqa/complain/complaintable");
            },
        },
    };
</script>

<style lang="less">
    .ef-sfcs-stat-table-footer {
        font-weight: bold;
        background-color: @table-selected-row-bg;
    }
</style>
<style scoped lang="less">
    .title {
        font-weight: bold;
    }

    .basebg {
        background-color: @base-bg-color;
    }
</style>