<template>
  <div v-bind:class="theme.mode == 'night' ? divmain : divmain2">
    <kanban-title :key="titleKey" :title="this.$t('kanbantitle')" :showTime="true" @stop="stopFreshData"
      @start="startFreshData" />

    <a-row :gutter="[20, 16]">
      <div style="display: flex ; justify-content: space-around;">
        <a-col :span="4" v-for="(item, i) in titlesums" :key="i">
          <v-title :title="item.title" :progress="item.per" :sum="item.sum" :key="i" :id="item.key">
          </v-title>
        </a-col>
      </div>
    </a-row>
    <br />
    <a-row :gutter="[20, 16]">
      <a-col :span="16">
        <a-spin :spinning="timerefresh ? loading : month_loading">
          <v-bar :title="monthtitle + '  ' + this.$t('month_main')" :data="month_main" :height="month_date_height"
            :position="'name*qty'" :yaxios="'qty'" :colortype="'type'" :series="[
              {
                quickType: 'line',
                position: 'name*total',
                color: 'totaltype',
              },
              {
                quickType: 'point',
                position: 'name*total',
                color: 'totaltype',
              },
            ]" :axisstyle="['name', 'qty', 'total']" :colorarray="[
              '#37ed3a',
              '#FF4040',
              '#33ccff',
              '#1ff2d2',
              '#FF7F00',
            ]" :guide="[
              {
                show: true,
                start: { name: '01', total: total_so },
                end: { name: '30', total: total_so },
                text: {
                  style: { fill: theme.mode === 'night' ? 'white' : 'black' },
                  content: this.$t('guidetxt') + '：' + total_so,
                },
              },
            ]" :padding="[14, 134, 24, 44]">
          </v-bar>
        </a-spin>
      </a-col>
      <a-col :span="8">
        <a-row>
          <a-col>
            <a-spin :spinning="timerefresh ? loading : year_loading">
              <v-bar :title="year + '  ' + this.$t('year_month')" :data="year_date" :height="year_date_height"
                :position="'name*qty'" :yaxios="'qty'" :padding="[24, 24, 24, 44]">
              </v-bar>
            </a-spin>
          </a-col>
        </a-row>
        <br />
        <a-row>
          <a-col>
            <a-spin :spinning="timerefresh ? loading : year_area_loading">
              <v-pie :title="year + '  ' + this.$t('year_area')" :data="year_area" :height="year_date_height"
                :padding="[0, 0, 0, 0]">
              </v-pie>
            </a-spin>
          </a-col>
        </a-row>
      </a-col>
    </a-row>
    <a-row :gutter="[20, 16]">
      <a-col :span="16">
        <a-spin :spinning="timerefresh ? loading : family_loading">
          <v-bar :title="monthtitle + '  ' + this.$t('month_family')" :data="family_main" :height="month_family_height"
            :position="'name*qty'" :yaxios="'qty'" :colortype="'type'" :colorarray="[
              '#37ed3a',
              '#FF4040',
              '#33ccff',
              '#1ff2d2',
              '#FF7F00',
            ]" :padding="[20, 14, 34, 44]" :legent="{ postion: 'top-right' }">
          </v-bar>
        </a-spin>
      </a-col>
      <a-col :span="8">
        <a-spin :spinning="timerefresh ? loading : area_loading">
          <v-bar :title="monthtitle + '  ' + this.$t('month_area')" :data="area_main" :height="month_family_height"
            :position="'name*qty'" :colortype="'type'" :yaxios="'qty'" :colorarray="[
              '#37ed3a',
              '#FF4040',
              '#33ccff',
              '#1ff2d2',
              '#FF7F00',
            ]" :padding="[24, 14, 34, 44]" :legent="{ postion: 'top-right' }">
          </v-bar>
        </a-spin>
      </a-col>
    </a-row>
  </div>
</template>

<script>
  import { mapState } from 'vuex'
  import KanbanTitle from '@/pages/kanban/common/kanban_com.vue'
  import { reqPost } from '@/services/base'
  import bgImg from './base_bg.jpg'
  import VTitle from '@/pages/kanban/common/title_com.vue'
  import VBar from '@/pages/kanban/common/bar_com.vue'
  import VPie from '@/pages/kanban/common/pie_com.vue'

  const DataSet = require('@antv/data-set')
  const fullHeight = window.screen.height

  export default {
    name: 'KanbanShip',
    i18n: require('./i18n'),
    data() {
      return {
        titleKey: 0,
        month_loadcount: 0,
        family_loadcount: 0,
        year_count: 0,
        year_area_count: 0,
        area_count: 0,

        bgImg: bgImg,
        month_date_height: fullHeight / 2 - 136,
        //    month_family_height: fullHeight / 2 - (this.theme.mode=='light'?324:335) ,
        year_date_height: (fullHeight / 2 - 250) / 2,

        month_family: [],
        year_date: [],
        month_main: [],
        family_main: [],
        year_area: [],
        area_main: [],

        divmain2: 'divmain2',
        divmain: 'divmain',

        regionnames: [],
        total_so: 0,

        loading: false,
        timerefresh: false,
      }
    },
    computed: {
      month_family_height() {
        //return (fullHeight / 2) - Number((this.theme.mode == 'light' ? '324' : '339'))
        return fullHeight / 2 - 339
      },
      ...mapState('setting', ['theme', 'lang']),
      refresh_loading() {
        if (
          this.area_loading ||
          this.year_area_loading ||
          this.month_loading ||
          this.year_loading ||
          this.family_loading
        ) {
          return false
        } else {
          return true
        }
      },
      area_loading() {
        return this.area_count > 0
      },
      year_area_loading() {
        return this.year_area_count > 0
      },
      month_loading() {
        return this.month_loadcount > 0
      },
      year_loading() {
        return this.year_count > 0
      },
      family_loading() {
        return this.family_loadcount > 0
      },
      titlesums() {
        return [
          {
            key: 'month_so',
            title: this.monthtitle + this.$t('month_so'),
            sum: 0,
            per: 100,
          },
          {
            key: 'month_io',
            title: this.monthtitle + this.$t('month_io'),
            sum: 0,
            per: 45,
          },
          {
            key: 'month_turnin',
            title: this.monthtitle + this.$t('month_turnin'),
            sum: 0,
            per: 23,
          },
          {
            key: 'month_gg',
            title: this.monthtitle + this.$t('month_gg'),
            sum: 0,
            per: 80,
          },
          {
            key: 'month_ship',
            title: this.monthtitle + this.$t('month_ship'),
            sum: 0,
            per: 40,
          },
          {
            key: 'year_ship',
            title: this.year + this.$t('year_ship'),
            sum: 0,
            per: 47,
          },
        ]
      },

      year() {
        return new Date().getFullYear()
      },
      monthtitle() {
        var res = ''
        var mon = new Date().getMonth() + 1
        var arr = [
          'Jan',
          'Feb',
          'Mar',
          'Apr',
          'May',
          'Jun',
          'Jul',
          'Aug',
          'Sept',
          'Oct',
          'Nov',
          'Dec',
        ]
        if (this.lang == 'us') {
          res = arr[Number(mon) - 1]
        } else {
          res = mon
        }
        return res
      },
      month() {
        return new Date().getMonth() + 1
      },
      monthlist() {
        var res = []
        for (var i = 1; i < 13; i++) {
          if (i < 10) {
            res.push({ key: '0' + i })
          } else {
            res.push({ key: i.toString() })
          }
        }
        return res
      },
      daylist() {
        var res = []
        var day = new Date(this.year, this.month, 0)
        for (var i = 1; i < day.getDate() + 1; i++) {
          if (i < 10) {
            res.push({ key: '0' + i })
          } else {
            res.push({ key: i.toString() })
          }
        }
        return res
      },
    },
    created() {
      // bus.$on("actived", msg => {
      //     var index = this.actived_title.findIndex(e => e.split("_")[0] == msg.split("_")[0])
      //     this.actived_title[index] = msg;
      //     this.actived_title.splice(index, 1, msg)

      // })
      var params = {
        prop: 'consts',
        key: 'SYSSOREGION',
      }
      reqPost('ERP_MRP', params).then((res) => {
        this.regionnames = res.data
      })
      this.handles()
    },
    watch: {
      lang() {
        this.handles()
      },
    },
    methods: {
      handles() {
        this.handle_year_ship()
        this.handle_month_main()
        this.handle_family_main()
        this.handle_area_ship()
        this.handle_area_main()
      },
      stopFreshData() {
        if (this.timer) {
          clearTimeout(this.timer)
        }
        this.timerefresh = false
      },
      startFreshData() {
        if (this.timer) {
          clearTimeout(this.timer)
        }
        this.timerefresh = true
        if (this.refresh_loading) {
          this.handles()
        } else {
          this.$message.error('Loading')
        }

        this.timer = setTimeout(this.startFreshData, 1000 * 60 * 3)
      },

      handle_year_ship() {
        var params_time = {
          prop: 'consts',
          key:
            'SYSSHIPBYMONTH(' +
            this.year +
            '-01-01 00:00:00)(' +
            this.year +
            '-12-31 23:59:59)',
        }

        this.year_count = 1
        var total_year_ship = 0
        reqPost('ERP_MRP', params_time).then((res) => {
          this.year_count--
          // this.year_date = res.data.map(e => {
          //     e.qty = Number(e.shipqty)
          //     e.name = e.billyearmonth.split('-')[1];
          //     total_year_ship = total_year_ship + Number(e.shipqty);
          //     return e;
          // })
          res.data.map((e) => {
            total_year_ship = total_year_ship + Number(e.shipqty)
          })
          this.year_date = this.monthlist.map((e) => {
            res.data.some((a) => {
              if (e.key === a.billyearmonth.split('-')[1]) {
                e.qty = Number(a.shipqty)
                e.name = a.billyearmonth.split('-')[1]
                return true
              } else {
                e.qty = 0
                e.name = e.key
              }
            })
            return e
          })
          this.titlesums.some((e) => {
            if (e.key == 'year_ship') {
              e.sum = total_year_ship
            }
          })
        })
      },

      handle_month_main() {
        var day = new Date(this.year, this.month, 0)
        var first =
          this.year +
          '-' +
          (Number(this.month) < 10 ? '0' + this.month : this.month) +
          '-01'
        var second = this.year + '-' + this.month + '-' + day.getDate()

        var params = {
          prop: 'consts',
          key:
            'ETDQTY(' +
            first +
            ' 00:00:00)(' +
            second +
            ' 23:59:59)(GTK)(billDate)',
        }
        var params_turnin = {
          prop: 'consts',
          key: 'ETDTURNIN(' + first + ' 00:00:00)(' + second + ' 23:59:59)(date)',
        }
        var params_io = {
          prop: 'consts',
          key: 'ETDINPUT(' + first + ' 00:00:00)(' + second + ' 23:59:59)(date)',
        }
        this.month_loadcount = 3
        var arr_gg = []
        var arr_etd = []
        var arr_ship = []
        var arr_turnin = []
        var arr_io = []
        var arr_totalgg = []
        var arr_totaletd = []
        var arr_totalship = []
        var arr_totalturnin = []
        var arr_totalio = []
        var total_gg = 0
        var total_etd = 0
        var total_ship = 0
        var total_turnin = 0
        var total_io = 0

        reqPost('ERP_MRP', params).then((res) => {
          this.month_loadcount--
          this.daylist
            .map((e) => {
              res.data.some((a) => {
                if (e.key === a.billdate.split('-')[2]) {
                  e.name = a.billdate.split('-')[2]
                  e.etdqty = a.etdqty
                  e.ggqty = a.ggqty
                  e.shipqty = a.shipqty
                  return true
                } else {
                  e.name = e.key
                  e.etdqty = 0
                  e.ggqty = 0
                  e.shipqty = 0
                }
              })
              return e
            })
            .map((e) => {
              total_gg = total_gg + Number(e.ggqty)
              total_ship = total_ship + Number(e.shipqty)
              total_etd = total_etd + Number(e.etdqty)
              arr_gg.push({
                name: e.name,
                qty: Number(e.ggqty),
                type: this.$t('gg'),
              })
              arr_ship.push({
                name: e.name,
                qty: Number(e.shipqty),
                type: this.$t('ship'),
              })
              arr_etd.push({
                name: e.name,
                qty: Number(e.etdqty),
                type: this.$t('so'),
              })

              arr_totalgg.push({
                name: e.name,
                total: total_gg,
                totaltype: this.$t('_gg'),
              })
              arr_totalship.push({
                name: e.name,
                total: total_ship,
                totaltype: this.$t('_ship'),
              })
              arr_totaletd.push({
                name: e.name,
                total: total_etd,
                totaltype: this.$t('_so'),
              })
            })

          reqPost('SFCS_RSBU', params_turnin).then((res) => {
            this.month_loadcount--
            var temp = []
            var total = 0
            res.data.map((e) => {
              if (e.key < first) {
                total = total + Number(e.qty)
              } else {
                temp.push({ key: e.key, qty: Number(e.qty) })
              }
            })
            this.daylist
              .map((e) => {
                temp.some((a) => {
                  if (e.key === a.key.split('-')[2]) {
                    e.name = a.key.split('-')[2]
                    e.qty = a.qty
                    return true
                  } else {
                    e.name = e.key
                    e.qty = 0
                  }
                })
                return e
              })
              .map((e) => {
                total_turnin = total_turnin + e.qty
                arr_turnin.push({
                  name: e.name || e.key,
                  qty: Number(e.qty),
                  type: this.$t('turnin'),
                })
                arr_totalturnin.push({
                  name: e.name || e.key,
                  total: total_turnin,
                  totaltype: this.$t('_turnin'),
                })
              })

            reqPost('SFCS_RSBU', params_io).then((res) => {
              this.month_loadcount--
              var temp_io = []
              res.data.map((e) => {
                if (e.key < first) {
                  //
                } else {
                  temp_io.push({ key: e.key, qty: Number(e.qty) })
                }
              })
              this.daylist
                .map((e) => {
                  temp_io.some((a) => {
                    if (e.key === a.key.split('-')[2]) {
                      e.qty = a.qty
                      e.name = a.key.split('-')[2]
                      return true
                    } else {
                      e.name = e.key
                      e.qty = 0
                    }
                  })
                  return e
                })
                .map((e) => {
                  total_io = total_io + Number(e.qty)
                  arr_io.push({
                    name: e.name,
                    qty: Number(e.qty),
                    type: this.$t('io'),
                  })
                  arr_totalio.push({
                    name: e.name,
                    total: total_io,
                    totaltype: this.$t('_io'),
                  })
                })
              this.titlesums.map((e) => {
                if (e.key === 'month_ship') {
                  e.sum = total_ship
                  e.per = (total_ship / total_etd) * 100
                } else if (e.key === 'month_so') {
                  e.sum = total_etd
                  this.total_so = total_etd
                } else if (e.key === 'month_gg') {
                  e.sum = total_gg
                  e.per = (total_gg / total_etd) * 100
                } else if (e.key === 'month_turnin') {
                  e.sum = total_turnin
                  e.per = (total_turnin / total_etd) * 100
                } else if (e.key === 'month_io') {
                  e.sum = total_io
                  e.per = (total_io / total_etd) * 100
                }
              })
              this.month_main = [
                ...arr_etd,
                ...arr_io,
                ...arr_turnin,
                ...arr_gg,
                ...arr_ship,
                ...arr_totaletd,
                ...arr_totalio,
                ...arr_totalturnin,
                ...arr_totalgg,
                ...arr_totalship,
              ]
            })
          })
        })
      },

      handle_family_main() {
        var day = new Date(this.year, this.month, 0)
        var first =
          this.year +
          '-' +
          (Number(this.month) < 10 ? '0' + this.month : this.month) +
          '-01'
        var second = this.year + '-' + this.month + '-' + day.getDate()

        var params_io = {
          prop: 'consts',
          key:
            'ETDINPUT(' +
            first +
            ' 00:00:00)(' +
            second +
            ' 23:59:59)(familyName)',
        }
        var params_ship = {
          prop: 'consts',
          key:
            'ETDSHIP(' +
            first +
            ' 00:00:00)(' +
            second +
            ' 23:59:59)(familyName)',
        }
        var params_gg_so = {
          prop: 'consts',
          key: 'ETDQTY(' + first + ' 00:00:00)(' + second + ' 23:59:59)',
        }
        var params_turnin = {
          prop: 'consts',
          key:
            'ETDTURNIN(' +
            first +
            ' 00:00:00)(' +
            second +
            ' 23:59:59)(familyName)',
        }

        var temp = []
        this.family_main = []
        this.family_loadcount = 4
        var arr_gg = []
        var arr_so = []
        var arr_ship = []
        var arr_turnin = []
        var arr_io = []
        reqPost('SFCS_RSBU', params_io).then((res) => {
          this.family_loadcount--
          arr_io = res.data.map((e) => {
            return { qty: Number(e.qty), type: this.$t('io'), name: e.key }
          })

          reqPost('SFCS_RSBU', params_turnin).then((res) => {
            this.family_loadcount--
            arr_turnin = res.data.map((e) => {
              return { qty: Number(e.qty), type: this.$t('turnin'), name: e.key }
            })

            reqPost('SFCS_RSBU', params_ship).then((res) => {
              this.family_loadcount--
              arr_ship = res.data.map((e) => {
                return { qty: Number(e.qty), type: this.$t('ship'), name: e.key }
              })
              reqPost('SFCS_RSBU', params_gg_so).then((res) => {
                this.family_loadcount--
                res.data.map((e) => {
                  arr_gg.push({
                    qty: Number(e.ggqty),
                    type: this.$t('gg'),
                    name: e.key,
                  })
                  arr_so.push({
                    qty: Number(e.qty),
                    type: this.$t('so'),
                    name: e.key,
                  })
                })
                temp = [
                  ...arr_so,
                  ...arr_io,
                  ...arr_turnin,
                  ...arr_gg,
                  ...arr_ship,
                ]
                this.family_main = temp
              })
            })
          })
        })
      },

      handle_area_ship() {
        var params = {
          prop: 'consts',
          key:
            'SYSSHIPBYREGION(' + this.year + '-01-01)(' + this.year + '-12-31)',
        }
        this.year_area_count++
        reqPost('ERP_MRP', params).then((res) => {
          this.year_area_count--
          var temp = res.data.map((e) => {
            e.qty = Number(e.shipqty)
            e.name = this.handle_regionname(e.region)
            return e
          })
          const dv2 = new DataSet.View().source(temp)
          dv2
            .transform({
              type: 'percent',
              field: 'qty',
              dimension: 'name',
              as: 'percent',
            })
            .transform({
              type: 'sort-by',
              fields: ['qty'], // 根据指定的字段集进行排序，与lodash的sortBy行为一致
              order: 'DESC', // 默认为 ASC，DESC 则为逆序
            })
          this.year_area = dv2.rows
        })
      },

      handle_area_main() {
        var day = new Date(this.year, this.month, 0)
        var first = this.year + '-' + this.month + '-01'
        var second = this.year + '-' + this.month + '-' + day.getDate()

        var area_ship = {
          prop: 'consts',
          key:
            'ETDQTY(' +
            first +
            ' 00:00:00)(' +
            second +
            ' 23:59:59)(GTK)(region)',
        }
        var params_turnin = {
          prop: 'consts',
          key:
            'ETDTURNIN(' + first + ' 00:00:00)(' + second + ' 23:59:59)(region)',
        }
        var params_io = {
          prop: 'consts',
          key:
            'ETDINPUT(' + first + ' 00:00:00)(' + second + ' 23:59:59)(region)',
        }
        this.area_count = 3
        reqPost('ERP_MRP', area_ship).then((res) => {
          this.area_count--
          var arr_gg = []
          var arr_etd = []
          var arr_ship = []
          var arr_turnin = []
          var arr_io = []

          res.data.map((e) => {
            arr_gg.push({
              name: this.handle_regionname(e.region),
              qty: Number(e.ggqty),
              type: this.$t('gg'),
            })
            arr_ship.push({
              name: this.handle_regionname(e.region),
              qty: Number(e.shipqty),
              type: this.$t('ship'),
            })
            arr_etd.push({
              name: this.handle_regionname(e.region),
              qty: Number(e.etdqty),
              type: this.$t('so'),
            })
          })
          reqPost('SFCS_RSBU', params_turnin).then((res) => {
            this.area_count--
            res.data.map((e) => {
              arr_turnin.push({
                name: this.handle_regionname(e.key),
                qty: Number(e.qty),
                type: this.$t('turnin'),
              })
            })
            reqPost('SFCS_RSBU', params_io).then((res) => {
              this.area_count--
              res.data.map((e) => {
                arr_io.push({
                  name: this.handle_regionname(e.key),
                  qty: Number(e.qty),
                  type: this.$t('io'),
                })
                this.area_main = [
                  ...arr_etd,
                  ...arr_io,
                  ...arr_turnin,
                  ...arr_gg,
                  ...arr_ship,
                ]
              })
            })
          })
        })
      },

      handle_regionname(element) {
        var res = ''

        this.regionnames.some((e) => {
          if (e.key === element) {
            res = this.lang == 'us' ? e.name_us : e.name
            return true
          }
        })
        return res === '' ? (this.lang == 'us' ? 'Other' : '其他') : res
      },
    },
    components: {
      VTitle,
      VBar,
      VPie,
      KanbanTitle,
    },
  }
</script>

<style lang="less" scoped>
  .divmain {
    z-index: -10;
    zoom: 1;
    background: url('./bg3.png') no-repeat;
    background-size: cover;
    -webkit-background-size: cover;
    -o-background-size: cover;
    background-position: center 0;
    padding: 20px;
  }

  .divmain2 {
    background: @layout-body-background;
    padding: 20px;
  }

  .extra-wrap {
    .extra-item {
      display: inline-block;
      margin-right: 24px;

      a:not(:first-child) {
        margin-left: 24px;
      }
    }
  }

  @media screen and (max-width: 992px) {
    .extra-wrap .extra-item {
      display: none;
    }
  }

  @media screen and (max-width: 576px) {
    .extra-wrap {
      display: none;
    }
  }

  .page {
    background-color: @layout-body-background;
  }
</style>