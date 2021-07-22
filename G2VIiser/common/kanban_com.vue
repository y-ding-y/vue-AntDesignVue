<template>
  <div v-bind:class="theme.mode == 'night' ? top : top2">
    <h1 v-bind:class="theme.mode == 'night' ? txtstyle : txtstyle2">
      SYS {{ year }}{{ month }} &nbsp;&nbsp;&nbsp; {{ title }}
    </h1>

    <span
      v-if="showTime"
      @click="toggleClock"
      style="position:absolute;right:24px;top:24px;cursor:pointer"
    >
      <a-icon
        :type="nTimer ? 'loading' : 'clock-circle'"
        style="margin-right:5px;"
      />{{ dateTime }}
    </span>
  </div>
</template>

<script>
import { mapState } from 'vuex'

export default {
  i18n: require('./i18n'),
  name: 'Title',
  props: {
    title: { type: String, default: 'Title String' },
    showTime: { type: Boolean, default: true },
    isRunning: { type: Boolean, default: false },
  },
  data() {
    return {
      dateTime: new Date().toStr(),
      nTimer: null,
      top: 'top',
      top2: 'top2',
      txtstyle: 'txtstyle',
      txtstyle2: 'txtstyle2',
    }
  },
  computed: {
    ...mapState('setting', ['theme', 'lang']),
    year() {
      return new Date().getFullYear() + (this.lang == 'us' ? ' ' : '年')
    },
    month() {
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
        res = mon + '月'
      }
      return res
    },
    monthlist() {
      var res = []
      for (var i = 1; i < 13; i++) {
        res.push({ key: i, value: i + '月' })
      }

      return res
    },
    yearlist() {
      var res = []
      var center = new Date().getFullYear()
      for (var i = Number(center) - 5; i < Number(center) + 5; i++) {
        res.push({ key: i, value: i + this.lang == 'us' ? 'Year' : '年' })
      }
      return res
    },
  },
  mounted() {
    if (this.isRunning) this.toggleClock()
  },
  destroyed() {
    clearInterval(this.nTimer)
  },
  methods: {
    toggleClock() {
      if (this.nTimer) {
        clearInterval(this.nTimer)
        this.nTimer = null
        this.$emit('stop')
      } else {
        this.nTimer = setInterval(() => {
          const now = new Date()
          this.dateTime = now.toStr()
        }, 300)
        this.$emit('start')
      }
    },
  },
}
</script>

<style lang="less" scoped>
.top {
  border: 0px solid;
  padding: 12px 24px 0 24px;
  margin: 0px 24px 12px 12px;
  text-align: center;
  color: white;
}

.top2 {
  border: 0px solid;
  padding: 12px 24px 0 24px;
  margin: 0px 24px 12px 12px;
  text-align: center;
  color: rgb(0, 0, 0);
}

.txtstyle {
  font-size: 200%;
  display: inline - block;
  letter-spacing: 5px;
  color: white;
}

.txtstyle2 {
  font-size: 200%;
  display: inline - block;
  letter-spacing: 5px;
  color: rgb(0, 0, 0);
}
</style>
