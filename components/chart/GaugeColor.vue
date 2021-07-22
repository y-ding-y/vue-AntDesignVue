<template>
  <v-chart
    :forceFit="forceFit"
    :height="height"
    :padding="padding"
    :data="dataSet"
    :scale="scaleSet"
    :animate="animate"
  >
    <v-coord
      :type="coordSet.type"
      :startAngle="coordSet.start"
      :endAngle="coordSet.end"
      :radius="coordSet.radius"
    />
    <v-axis
      :dataKey="axisSet.dataKey"
      :zIndex="axisSet.zIndex"
      :line="axisSet.line"
      :label="axisSet.label"
      :subTickCount="axisSet.subTickCount"
      :subTickLine="axisSet.subTick"
      :tickLine="axisSet.tick"
      :grid="axisSet.grid"
    />
    <v-axis dataKey="1" :show="false" />
    <v-series
      gemo="point"
      shape="pointer"
      :position="gemoSet.position"
      :color="gemoSet.color"
      :active="gemoSet.active"
    />
    <!--eslint-disable-next-line-->
    <v-guide
      v-for="g in arcGuides"
      type="arc"
      :key="g.start.toString() + g.end.toString()"
      :zIndex="g.zIndex"
      :start="g.start"
      :end="g.end"
      :vStyle="g.style"
    />
    <v-guide
      type="html"
      :position="labelGuideSet.position"
      :html="labelGuideHtml"
    />
  </v-chart>
</template>

<script>
import { registerShape } from 'viser-vue'
import { isDef } from '@/utils/util'
import merge from 'deepmerge'

registerShape('point', 'pointer', {
  draw(cfg, container) {
    let point = cfg.points[0] // 获取第一个标记点
    point = this.parsePoint(point)
    const center = this.parsePoint({
      // 获取极坐标系下画布中心点
      x: 0,
      y: 0,
    })
    // 绘制指针
    container.addShape('line', {
      attrs: {
        x1: center.x,
        y1: center.y,
        x2: point.x,
        y2: point.y,
        stroke: cfg.color,
        lineWidth: 3,
        lineCap: 'round',
      },
    })
    return container.addShape('circle', {
      attrs: {
        x: center.x,
        y: center.y,
        r: 4,
        stroke: cfg.color,
        lineWidth: 2,
        fill: '#fff',
      },
    })
  },
})

export default {
  props: {
    //圖表高度
    height: {
      type: Number,
      default: 120,
    },
    //寬度自適應
    forceFit: { type: Boolean, default: true },
    //動畫
    animate: { type: Boolean, default: false },
    //圖表內留白
    padding: {
      type: [Number, Array],
      default: () => [10, 10, 10, 10],
    },
    //數據
    data: {
      default: () => [{ value: 10 }],
    },
    //度量
    scaleDef: {
      type: Object,
      default: () => ({
        min: 0,
        max: 100,
        tickInterval: 20,
        nice: false,
      }),
    },
    scale: {
      default: () => [{ dataKey: 'value' }],
    },
    //坐標系
    coordDef: {
      type: Object,
      default: () => ({
        type: 'polar',
        start: -202.5,
        end: 22.5,
        radius: 0.95,
      }),
    },
    coord: {
      type: Object,
      default: () => ({}),
    },

    //坐標軸數值
    axisDef: {
      type: Object,
      default: () => ({
        zIndex: 2,
        line: null,
        label: {
          offset: -10,
          textStyle: {
            fontSize: 12,
            textAlign: 'center',
            textBaseline: 'middle',
          },
        },
        //坐標軸子刻度
        subTick: {
          length: -12,
          stroke: '#999',
          strokeOpacity: 1,
        },
        subTickCount: 4,
        //坐標軸刻度
        tick: { length: -14, stroke: '#999', strokeOpacity: 1 },
        grid: null,
      }),
    },
    axis: {
      type: Object,
      default: () => ({}),
    },
    //輔助元素設置
    //  背景弧形設置值模板
    backGuideDef: {
      type: Object,
      default: () => ({
        top: false,
        zIndex: 0,
        start: 0,
        end: 100,
        style: { stroke: '#FA541C', lineWidth: 10 },
      }),
    },
    //  背景弧形設置
    backGuides: {
      default: () => [{}],
    },
    //  前景弧形設置值模板
    frontGuideDef: {
      type: Object,
      default: () => ({
        top: true,
        zIndex: 1,
        start: 30,
        end: 40,
        style: { stroke: '#2FC25B', lineWidth: 10 },
      }),
    },
    //  前景弧形設置
    frontGuides: {
      default: () => [],
    },
    //標籤位置和格式
    labelGuideDef: {
      default: () => ({
        position: ['50%', '94%'],
        fixed: 1, //小數位
        padLen: 4, //填充長度
        padChar: '0', //填充字符
        name: '指標', //指標名稱
        separator: ' , ',
        html: `<p style="width:auto;text-align: center;font-size: 12px;margin: 0;">
          {name}<br/><span style="font-size:16px;font-weight:bold;padding:0 2px;line-height:18px;">{value}</span>
        </p>`,
      }),
    },
    labelGuide: {
      default: () => ({}),
    },
    gemoDef: {
      default: () => ({ color: '#8C8C8C', active: false }),
    },
    gemo: {
      default: () => ({}),
    },
  },
  data() {
    return {
      arcGuides: [],
      dataSet: [{ value: 0 }, { value: 10 }],
    }
  },

  computed: {
    scaleSet: function() {
      return this.scale.map((e) => merge(this.scaleDef, e))
    },
    coordSet: function() {
      return merge(this.coordDef, this.coord)
    },
    axisSet: function() {
      const scale = this.scaleSet[0]
      return merge.all([this.axisDef, { dataKey: scale.dataKey }, this.axis])
    },
    labelGuideSet: function() {
      return merge(this.labelGuideDef, this.labelGuide)
    },
    labelGuideHtml: function() {
      const key = this.scaleSet[0].dataKey
      const lg = this.labelGuideSet
      return lg.html.replace('{name}', lg.name).replace(
        '{value}',
        this.dataSet
          .map((e) => {
            let val = isDef(e[key]) ? e[key] : null
            val = isDef(val) && isDef(lg.fixed) ? val.toFixed(lg.fixed) : val
            val = isDef(val) ? val : null
            return isDef(val) && isDef(lg.padLen) && isDef(lg.padChar)
              ? val.padStart(lg.padLen, lg.padChar)
              : val
          })
          .join(this.labelGuideSet.separator)
      )
    },
    gemoSet: function() {
      return merge.all([
        this.gemoDef,
        { position: this.scaleSet[0].dataKey + '*1' },
        this.gemo,
      ])
    },
  },
  watch: {
    data: function() {
      this.$data.dataSet = isDef(this.data) ? this.data : []
    },
    backGuides: function() {
      this.mergeGuides()
    },
    frontGuides: function() {
      this.mergeGuides()
    },
  },
  methods: {
    setData() {
      if (this.timer) {
        clearTimeout(this.timer)
      }
      this.dataSet[0].tm = Math.floor(Math.random() * 100 * 10) / 10
      this.timer = setTimeout(this.setData, 1000)
    },
    mergeGuides() {
      const radius = this.coordSet.radius
      this.arcGuides = [
        ...this.backGuides.map((e) =>
          merge(this.backGuideDef, e, { arrayMerge: (a, b) => b })
        ),
        ...this.frontGuides.map((e) =>
          merge(this.frontGuideDef, e, { arrayMerge: (a, b) => b })
        ),
      ].map((e) => {
        e.start = Array.isArray(e.start) ? e.start : [e.start, radius]
        e.end = Array.isArray(e.end) ? e.end : [e.end, radius]
        return e
      })
    },
  },
  mounted() {
    //this.timer = setTimeout(this.setData, 0)
    this.mergeGuides()
  },
  beforeDestroy() {
    if (this.timer) {
      clearTimeout(this.timer)
    }
  },
}
</script>
