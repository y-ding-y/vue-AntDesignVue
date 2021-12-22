<template>
  <div>
    <ef-main :isMainPage="true"  ref="root">
      <a-tabs default-active-key="upload">
        <a-tab-pane key="upload" :tab="$t('upload')"><v-upload /></a-tab-pane>
        <a-tab-pane key="search" :tab="$t('search')"><v-search /></a-tab-pane>
      </a-tabs>
    </ef-main>
  </div>
</template>

<script>
import EfMain from '@/components/page/main'
import { merge } from '@/utils/util'
import VSearch from './searchetd.vue'
import VUpload from './upload.vue'

const i18n = require('./i18n')

export default {
  name: 'efetd',
  i18n: merge(require('@/i18n'), i18n),
  components: { EfMain, VSearch, VUpload },
}
</script>

<style>
.a-upload {
  width: 120px;
  height: 35px;
}
</style>
