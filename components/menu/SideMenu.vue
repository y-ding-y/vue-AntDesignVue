<template>
  <a-layout-sider
    :theme="sideTheme"
    :class="['side-menu', 'beauty-scroll', isMobile ? null : 'shadow']"
    width="256px"
    :collapsible="collapsible"
    v-model="collapsed"
    :trigger="null"
  >
    <div :class="['logo', theme]">
      <router-link to="/">
        <header-icon />
        <h1>{{ systemName }}</h1>
        <sup>{{ systemVersion }}</sup>
      </router-link>
    </div>
    <i-menu
      :theme="theme"
      :collapsed="collapsed"
      :options="menuData"
      @select="onSelect"
      class="menu"
    />
  </a-layout-sider>
</template>

<script>
import IMenu from './menu'
import HeaderIcon from '@/layouts/header/HeaderIcon'
import { mapState } from 'vuex'
export default {
  name: 'SideMenu',
  components: { IMenu, HeaderIcon },
  props: {
    collapsible: {
      type: Boolean,
      required: false,
      default: false,
    },
    collapsed: {
      type: Boolean,
      required: false,
      default: false,
    },
    menuData: {
      type: Array,
      required: true,
    },
    theme: {
      type: String,
      required: false,
      default: 'dark',
    },
  },
  computed: {
    sideTheme() {
      return this.theme == 'light' ? this.theme : 'dark'
    },
    ...mapState('setting', ['isMobile', 'systemName', 'systemVersion']),
  },
  methods: {
    onSelect(obj) {
      this.$emit('menuSelect', obj)
    },
  },
}
</script>

<style lang="less" scoped>
@import 'index';
</style>
