<template>
  <div>
    <a-card v-bind:class="theme.mode == 'night' ? cardsum : cardsum2">
      <ef-chart class="basebg" :title="title" :titleStyle="theme.mode == 'night' ? titlestyle : titlestyle2"
        :height="height" :padding="padding" :scale="scaleSet" :series="
          [
            ...[
              {
                quickType: 'bar',
                position: position,
                color: colortype,
                adjust: [
                  {
                    type: 'dodge',
                    marginRatio: 1 / 32,
                  },
                ],
              },
            ],
            ...series,
          ].map((e) => {
            e.color = [e.color, colorarray]
            return e
          })
        " :axis="chartstyle" :data="data" :guide="guide" :legent="legent">
      </ef-chart>
    </a-card>
  </div>
</template>

<script>
  import { mapState } from 'vuex'
  import Chart from '@/components/chart/Chart'
  export default {
    props: {
      title: String,
      data: Array,
      height: Number,
      position: String,
      yaxios: String,
      colortype: { type: String, default: '' },
      series: { type: Array, default: () => [] },
      axisstyle: { type: Array, default: () => ['name', 'qty'] },
      padding: { type: Array, default: () => [24, 44, 44, 44] },
      colorarray: {
        type: Array,
        default: () => ['#37ed3a', '#FF4040', '#f6ff45', '#1ff2d2'],
      },
      scaleDef: {
        type: Array,
        default: () => [{ dataKey: 'name', type: 'cat' }],
      },
      scale: { type: Array, default: () => [] },
      guide: { type: Array, default: () => [{}] },
      legent: { type: Object, default: () => ({ position: 'right' }) },
    },
    computed: {
      scaleSet() {
        return [...this.scale, this.scaleDef]
      },
      ...mapState('setting', ['theme']),
      chartstyle() {
        return this.axisstyle.map((e) => {
          return {
            dataKey: e,
            label: {
              textStyle: {
                fill: this.theme.mode == 'night' ? 'white' : 'black',
                fontWeight: 400,
              },
            },
          }
        })
      },
    },
    data() {
      return {
        titlestyle: { textAlign: 'center', color: 'white', fontSize: '20px' },
        titlestyle2: { textAlign: 'center', color: 'black', fontSize: '20px' },

        cardsum: 'cardsum',
        cardsum2: 'cardsum2',
      }
    },
    components: {
      EfChart: Chart,
    },
  }
</script>

<style lang="less" scoped>
  .cardsum {
    background: rgba(80, 80, 110, 0.452);
    border: 0px solid;
  }

  .cardsum2 {
    background: rgb(255, 255, 255);
    border: 0px solid;
  }
</style>