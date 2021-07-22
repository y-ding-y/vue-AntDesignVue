<template>
  <a-card
    :bordered="false"
    :body-style="{
      padding: '12px 24px 0 24px',
      margin: '0px 24px 12px 12px',
      textAlign: 'center',
    }"
    ><h1 style="font-size:200%;display:inline-block;letter-spacing:5px;">
      {{ title }}
    </h1>
    <span
      v-if="showTime"
      @click="toggleClock"
      style="position:absolute;right:24px;top:24px;cursor:pointer"
      ><a-icon
        :type="nTimer ? 'loading' : 'clock-circle'"
        style="margin-right:5px;"
      />{{ dateTime }}
    </span>
  </a-card>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
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
    }
  },
  computed: {
    ...mapGetters('setting', ['lang']),
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
@import 'index';
</style>
