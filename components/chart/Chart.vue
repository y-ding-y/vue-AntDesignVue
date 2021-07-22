<template>
  <div class="default">
    <h4 v-if="title" :style="titleStyle">{{ title }}</h4>
    <div class="chart">
      <v-chart :forceFit="forceFit" :height="height" :data="data" :padding="padding" :scale="scale">
        <v-tooltip />
        <v-axis v-for="(s, i) in axisSet" :key="'info' + i" v-bind="s" />
        <v-legend v-if="legentSet.show" :position="legentSet.position" />
        <v-series v-for="(s, i) in serieSet" :key="i" v-bind="s" />
        <v-guide v-for="(item, i) in guideSet" :key="'guide' + i" v-bind="item" />
      </v-chart>
    </div>
  </div>
</template>

<script>
  import merge from 'deepmerge'

  export default {
    name: 'Chart',
    props: {
      title: String,
      titleStyle: { type: [String, Object], default: '' },
      forceFit: { type: Boolean, default: true },
      height: { type: Number, default: 240 },
      padding: { type: [Array, Number], default: () => [24, 42] },
      //度量
      scaleDef: {
        type: Object,
        default: () => ({
          type: 'linear', // 度量的类型 連續:linear、log、pow、time,離散:cat,timeCat
          nice: true, // 默认为 true，用于优化数值范围，使绘制的坐标轴刻度线均匀分布。例如原始数据的范围为 [3, 97]，如果 nice 为 true，那么就会将数值范围调整为 [0, 100]
          //range: {array}, // 数值范围区间，即度量转换的范围，默认为 [0, 1]
          //alias: {string}, // 为数据属性定义别名，用于图例、坐标轴、tooltip 的个性化显示
          //ticks: {array}, // 存储坐标轴上的刻度点文本信息
          tickCount: 5, // 坐标轴上刻度点的个数，不同的度量类型对应不同的默认值
          //formatter: {function}, // 回调函数，用于格式化坐标轴刻度点的文本显示，会影响数据在坐标轴、图例、tooltip 上的显示
        }),
      },
      scale: {
        type: Array,
        default: () => [
          {
           
            //  dataKey: 'y',
            //  title: '时间',
            //  alias: 'Input',
            //  formatter: eval("(e) => (e/100).toFixed(2) + '%'"),
          },
        ],
      },

      axisDef: {
        type: Object,
        default: () => ({ show: true }),
      },
      axis: {
        type: [Object, Array],
        default: () => ({}),
      },

      legentDef: {
        type: Object,
        default: () => ({ show: true, position: 'top-right' }),
      },

      legent: {
        type: Object,
        default: () => ({}),
      },

      tooltip: {
        type: Array,
        default: () => [],
      },

      /**
       * quickType	说明
       * line:线型图，点按照 x 轴连接成一条线，构成线图
       *  smoothLine	平滑线型图
       *  dashLine	虚线型图
       *  stackLine	堆积线型图
       * area	面积图, 填充线图跟坐标系之间构成区域图，也可以指定上下范围。
       *  smoothArea	平滑面积
       *  stackArea	堆积面积
       * bar	柱状图
       *  stackBar	堆叠柱状图
       *  dodgeBar	分组柱状图
       * point	点，用于点图的构建
       * funnel	对称漏斗图
       *  pyramid	漏斗图
       * schema	k 线图，箱型图。
       *  box	箱型图
       *  candle	烛形图
       *  polygon	多边形，可以用于构建热力图、地图等图表类型。，todo
       * contour	todo
       * heatmap	热力图
       * edge	边，适用于树图、流程图、关系图。
       * errorBar	todo
       * jitterPoint	扰动点图
       * path	路径，无序的点连接而成的一条线。
       * interval	使用矩形或者弧形，用面积来表示大小关系的图形，一般构成柱状图、饼图等图表。
       *  stackInterval	堆叠
       *  dodgeInterval	分组
       * sector	todo
       */
      serieDef: {
        type: Object,
        default: () => ({
          quickType: 'bar',
          position: 'x*y',
        }),
      },
      series: {
        type: [Object, Array],
        default: () => [{}],
      },
      guide: {
        type: Array,
        default: () => [
          {
            //   show: false,
            //   start: {},
            //   end: {}
          },
        ],
      },
      guideDef: {
        type: Object,
        default: () => ({
          type: 'line',
          top: true,
          text: {
            position: 'start',
            style: {
              fill: 'white',
              fontSize: 15,
              fontWeight: 500,
              align: 'left',
            },
            content: '輔助线',
            offsetX: -20,
            offsetY: 20,
          },
          lineStyle: {
            stroke: 'green',
            lineWidth: 3,
            lineDash: [5, 5],
          },
        }),
      },

      data: {
        type: [Object, Array],
        default: () => [],
      },
    },
    computed: {
      axisSet: function () {
        const axises = Array.isArray(this.axis) ? this.axis : [this.axis]
        return axises.map((e) => merge(this.axisDef, e))
      },
      legentSet: function () {
        return merge(this.legentDef, this.legent)
      },
      serieSet: function () {
        return this.series.map((e) => merge(this.serieDef, e))
      },
      guideSet: function () {
        return this.guide.map((e) => merge(this.guideDef, e))
      },
      scaleSet: function () {
        return this.scale.map((e) => merge(this.scaleDef, e))
      },
    },
    mounted: function () { },
  }
</script>

<style scoped lang="less">
  .default {
    position: relative;
  }
</style>