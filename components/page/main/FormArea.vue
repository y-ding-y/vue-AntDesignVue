<template>
  <div :class="autoHide ? 'form-area form-area-hover' : 'form-area'" ref="root" @mouseenter="mouseenter"
    @mouseleave="mouseleave">
    <div class="select-root" ref="selectRoot"></div>
    <div class="form-item" v-show="item.value || mouseon || !autoHide" :key="index" v-for="(item, index) in formItems">
      <!--range-->
      <div v-if="item.type === 'range'" :class="['title', { active: item.value }]">
        <slot v-if="item.slots && item.slots.title" :name="item.slots.title"></slot>
        <template v-else>{{ getTitle(item, lang) }}</template>
        <a-range-picker :format="item.format" v-model="item.value"
          :disabled-date="(current) => disabledRangeDate(current, item)"
          @change="(dates, datesStr) => onRangeChange(dates, datesStr, item)"
          @open-change="(open) => onRangeOpenChange(open, item)"
          @calendarChange="(dates) => onRangeCalendarChange(dates, item)" :ranges="ranges" class="range-picker"
          :size="size" v-bind="item.attrs" :get-calendar-container="() => $refs.root">
          <a-icon slot="suffixIcon" type="calendar" />
        </a-range-picker>
      </div>
      <!--datetime-->
      <div v-else-if="item.type === 'datetime'" :class="['title', 'datetime', { active: item.value }]">
        <slot v-if="item.slots && item.slots.title" :name="item.slots.title"></slot>
        <template v-else>{{ getTitle(item, lang) }}</template>
        <a-date-picker :format="item.format" v-model="item.value"
          @change="(date, dateStr) => onCalendarChange(date, dateStr, item)"
          @open-change="(open) => onCalendarOpenChange(open, item)" class="datetime-picker" style="min-width:104px;"
          :size="size" show-time v-bind="item.attrs" :get-calendar-container="() => $refs.root" />
      </div>
      <!--date-->
      <div v-else-if="item.type === 'date'" :class="['title', { active: item.value }]">
        <slot v-if="item.slots && item.slots.title" :name="item.slots.title"></slot>
        <template v-else>{{ getTitle(item, lang) }}</template>
        <a-date-picker :format="item.format" v-model="item.value" @change="onDateChange(item)" class="date-picker"
          :size="size" v-bind="item.attrs" :get-calendar-container="() => $refs.root" />
      </div>
      <div v-else-if="item.type === 'month'" :class="['title', { active: item.value }]">
        <slot v-if="item.slots && item.slots.title" :name="item.slots.title"></slot>
        <template v-else>{{ getTitle(item, lang) }}</template>
        <a-month-picker :format="item.format" v-model="item.value" :placeholder="$ta('plsSelect| |time')"
          @change="(time, timeStr) => onCalendarChange(time, timeStr, item)"
          @open-change="(open) => onCalendarOpenChange(open, item)" class="date-picker" :size="size" v-bind="item.attrs"
          :get-calendar-container="() => $refs.root" />
      </div>
      <!--time-->
      <div v-else-if="item.type === 'time'" :class="['title', { active: item.value }]">
        <slot v-if="item.slots && item.slots.title" :name="item.slots.title"></slot>
        <template v-else>{{ getTitle(item, lang) }}</template>
        <a-time-picker :format="item.format" v-model="item.value" :placeholder="$ta('plsSelect| |time')"
          @change="(time, timeStr) => onCalendarChange(time, timeStr, item)"
          @open-change="(open) => onCalendarOpenChange(open, item)" class="time-picker" :size="size" v-bind="item.attrs"
          :get-popup-container="() => $refs.root" />
      </div>
      <!--select-->
      <div v-else-if="item.type === 'select'" :class="['title', { active: item.value !== undefined }]">
        <slot v-if="item.slots && item.slots.title" :name="item.slots.title"></slot>
        <template v-else>{{ getTitle(item, lang) }}</template>
        <a-select :options="getOptions(item, lang)" v-model="item.value" @change="onSelectChange(item)"
          v-bind="getSelectAttrs(item)" :filterOption="onFilterOption" :maxTagCount="getMaxTagCount(item)"
          :get-popup-container="() => $refs.selectRoot">
        </a-select>
      </div>
      <!--switch-->
      <div v-else-if="item.type === 'switch'" :class="['title', { active: item.value !== undefined }]">
        <slot v-if="item.slots && item.slots.title" :name="item.slots.title"></slot>
        <template v-else>{{ getTitle(item, lang) }}</template>
        <a-switch @change="onSwitchChange(item)" class="switch" v-model="item.value" :size="size"
          :checked-children="item.checked || $t('y')" :un-checked-children="item.unchecked || $t('n')"
          v-bind="item.attrs" />
        <a-icon v-if="item.value !== undefined" class="close" @click="(e) => onCloseClick(e, item)" type="close-circle"
          theme="filled" />
      </div>
      <!--input-->
      <div v-else-if="item.type === 'input'" :class="['title', { active: item.value !== undefined }]">
        <slot v-if="item.slots && item.slots.title" :name="item.slots.title"></slot>
        <template v-else>{{ getTitle(item, lang) }}</template>
        <a-input v-model="item.value" :size="size" v-bind="getInputAttrs(item)" />
      </div>
      <!--textarea-->
      <div v-else-if="item.type === 'textarea'" :class="['title', { active: item.value !== undefined }]">
        <slot v-if="item.slots && item.slots.title" :name="item.slots.title"></slot>
        <template v-else>{{ getTitle(item, lang) }}</template>
        <a-textarea v-model="item.value" :size="size" v-bind="getTextareaAttrs(item)" />
      </div>
      <!--popinput poptxt-->
      <div v-else-if="item.type === 'popinput' || item.type === 'poptextarea'"
        :class="['title', { active: item.value }]">
        <a-popover @visible-change="onVisibleChange(item, index)" v-model="item.visible" placement="bottom"
          :trigger="['click']" :get-popup-container="() => $refs.root">
          <slot v-if="item.slots && item.slots.title" :name="item.slots.title"></slot>
          <template v-else>{{ getTitle(item, lang) }}</template>
          <div class="value" v-if="item.value">
            :&nbsp;&nbsp;{{
            item.format && typeof item.format === 'function'
            ? item.format(item.value)
            : item.value
            }}
          </div>
          <a-icon v-if="!item.value" class="icon-down" type="down" />
          <div class="operations" slot="content">
            <a-button @click="onCancel(item)" class="btn" :size="size" type="link">{{ $t('cancle') }}</a-button>
            <a-button @click="onConfirm(item)" class="btn" :size="size" type="primary">{{ $t('confirm') }}</a-button>
          </div>
          <div class="form-overlay" slot="title">
            <a-input v-if="item.type === 'popinput'" :id="`${formIdPrefix}${index}`" :allow-clear="true"
              @keyup.esc="onCancel(item)" @keyup.enter="onConfirm(item)" v-model="item.value" :size="size"
              v-bind="item.attrs" />
            <a-textarea v-if="item.type === 'poptextarea'" :id="`${formIdPrefix}${index}`" :allow-clear="true"
              @keyup.esc="onCancel(item)" v-model="item.value" :size="size" v-bind="item.attrs" />
          </div>
        </a-popover>
        <a-icon v-if="item.value" @click="(e) => onCloseClick(e, item)" class="close" type="close-circle"
          theme="filled" />
      </div>
    </div>
    <a-button v-if="$listeners.submit && formItems && formItems.length > 0" :size="size" @click="onSubmit"
      :loading="loading" type="primary">{{ $t('search') }}</a-button>
  </div>
</template>

<script>
  import { mapState } from 'vuex'
  import { reqPost } from '@/services/base'
  import moment from 'moment'
  import { merge, isEmpty } from '@/utils/util'

  const i18n = merge(require('@/i18n'), require('./i18n'))
  export default {
    name: 'FormArea',
    //i18n: merge(require('@/i18n'), require('./i18n')),
    props: {
      id: String,
      i18n: { type: Object, default: () => ({}) },
      loading: { type: Boolean, default: false },
      size: { type: String, default: 'small' },
      items: {
        type: Array,
        default: () => [],
      },
      autoHide: { type: Boolean, default: false },
      openSubmit: { type: Boolean, default: false },
    },
    created() {
      const { messages } = merge(i18n, this.i18n ?? {})
      Object.keys(messages).forEach((lang) => {
        this.$i18n.mergeLocaleMessage(lang, messages[lang])
      })
      this.iniFormItems()
    },
    mounted() {
      this.openSubmit &&
        this.$emit('submit', this.getFormValues(), this.getFormOptions())
    },
    watch: {},
    data() {
      return {
        formValues: {},
        formOptions: [],
        formItems: [], //  [{key,dataIndex,options},...]
        optionSet: {},
        allParaData: [], //  [ {key:paraKey,data:[{key,title,title_us,title_cn}...]},...]
        allParaMap: {}, //  { paraKey:paraMap,...}
        allParaSet: [], //  [ { key: paraKey, paras: paraAtt },...]
        mouseon: false,
        mouseTimer: null,
      }
    },
    computed: {
      ...mapState('setting', ['lang']),
      formIdPrefix() {
        return this.id + '-ipt-'
      },
      ranges() {
        return {
          [this.$t('today')]: [moment(), moment()],
          [this.$t('thisweek')]: [
            moment().startOf('week'),
            moment().endOf('week'),
          ],
          [this.$t('thismonth')]: [
            moment().startOf('month'),
            moment().endOf('month'),
          ],
        }
      },
    },
    methods: {
      iniRangeValue(value) {
        const qty =
          value === 'yestoday' || value === 'lastweek' || value === 'lastmonth'
            ? -1
            : value === 'tomorrow' ||
              value === 'nextweek' ||
              value === 'nextmonth'
              ? 1
              : 0
        const unit =
          value === 'month' || value === 'lastmonth' || value === 'nextmonth'
            ? 'month'
            : value === 'week' || value === 'lastweek' || value === 'nextweek'
              ? 'week'
              : 'day'
        return value === 'today' ||
          value === 'yestoday' ||
          value === 'tomorrow' ||
          value === 'week' ||
          value === 'lastweek' ||
          value === 'nextweek' ||
          value === 'month' ||
          value === 'lastmonth' ||
          value === 'nextmonth'
          ? [
            moment()
              .add(qty, unit)
              .startOf(unit),
            moment()
              .add(qty, unit)
              .endOf(unit),
          ]
          : value && value[0]
            ? [moment(value[0]), moment(value[1])]
            : [null, null]
      },
      iniFormItems() {
        const paras = []
        const maps = {}
        this.formItems = this.items.map((item) => {
          let {
            key,
            type,
            dataIndex,
            value,
            paraKey,
            paraMap,
            paraAtt,
            ...attrs
          } = item
          item.format = this.getFormat(item)
          item.dataIndex = dataIndex = dataIndex || key
          item.attrs = attrs
          if (item.type === 'range') {
            item.value = this.iniRangeValue(value)
          } else if (['date', 'month', 'week', 'datetime'].includes(type)) {
            item.value = moment(value)
          }
          if (paraKey && !paras.find((e) => e.key == paraKey)) {
            paras.push({ key: paraKey, paras: paraAtt })
            if (paraMap) {
              maps[paraKey] = paraMap
            }
          }
          return item
        })
        this.allParaMap = maps
        this.allParaSet = paras
        this.loadParas()
      },
      getTitle(item, lang) {
        let title = ''
        if (item.title !== '' && item.title !== null) {
          title = item['title_' + lang] || item.title || this.$t(item.key)
        } else {
          title = ''
        }
        title += item.type != 'input' && title ? ':' : ''
        return title
      },
      getMaxTagCount: (item) => {
        return item.maxTagCount ??
          (Array.isArray(item.value) && item.value.length > 1)
          ? 0
          : 1
      },
      getOptions(item, lang) {
        const paraData = this.allParaData.find(
          (e) => e && item.paraKey && e.key === item.paraKey
        )
        const data = paraData && paraData.data ? paraData.data : []
        const options = (item.options || []).concat(data)
        const rtn = options.map((e) => {
          if (typeof e === 'string') {
            return { key: e, title: this.$t(e) }
          } else {
            return {
              key: e.key,
              title: e['title_' + lang] || e.title || this.$t(e.key),
            }
          }
        })
        return rtn
      },
      getSelectAttrs(item) {
        const def = {
          class: 'select',
          slot: 'content',
          size: 'small',
          allowClear: true,
          showArrow: true,
          showSearch: true,
          dropdownMatchSelectWidth: false,
          placeholder: this.$t('plsSelect') + '...',
        }
        return { ...def, ...(item.attrs || {}) }
      },
      getInputAttrs(item) {
        const def = {
          class: 'select',
          slot: 'content',
          size: 'small',
          allowClear: true,
          placeholder: this.$t('plsInput') + '...',
        }
        return { ...def, ...(item.attrs || {}) }
      },
      getTextareaAttrs(item) {
        const def = {
          class: 'select',
          slot: 'content',
          size: 'small',
          allowClear: true,
          placeholder: this.$t('plsInput') + '...',
        }
        return { ...def, ...(item.attrs || {}) }
      },
      onFilterOption(input, option) {
        const {
          key,
          data: {
            props: { title },
          },
        } = option
        const rtn = (key + '*' + title)
          .toLowerCase()
          .includes(input.toLowerCase())
        return rtn
      },
      loadParas() {
        const paras = this.allParaSet
        const param = this.allParaMap
        if (paras.length == 0) return
        reqPost('PARA_SET', { paras }).then((res) => {
          const arrData = []
          res.data.forEach((item) => {
            const { key, data } = item
            const map = param[key] || {}
            let options = []
            if (Array.isArray(data)) {
              options = data.map((option) => {
                if (!isEmpty(map)) {
                  if (map.key && option[map.key]) {
                    option.key = option[map.key]
                  }
                  if (map.title && option[map.title]) {
                    option.title = option[map.title]
                  }
                  if (map.title_us && option[map.title_us]) {
                    option.title_us = option[map.title_us]
                  }
                  if (map.title_cn && option[map.title_cn]) {
                    option.title_cn = option[map.title_cn]
                  }
                } else {
                  if (!option.key && option.id) {
                    option.key = option.id
                  }
                  if (!option.title && option.name) {
                    option.title = option.name
                  }
                }
                return option
              })
            } else if (data) {
              const keys = Object.keys(data)
              options = keys.map((k) => ({ key: k, title: data[k] }))
            }
            arrData.push({ key, data: options })
          })
          this.allParaData = arrData
        })
      },
      onSubmit() {
        this.$emit('submit', this.getFormValues(), this.getFormOptions())
      },
      onCloseClick(e, item) {
        e.preventDefault()
        e.stopPropagation()
        item.value = undefined
        const { backup, value } = item
        if (backup !== value) {
          this.backupAndEmitChange(item)
        }
      },
      onCancel(item) {
        item.value = item.backup
        item.visible = false
      },
      onConfirm(item) {
        const { backup, value } = item
        item.visible = false
        if (backup !== value) {
          this.backupAndEmitChange(item)
        }
      },
      onSwitchChange(item) {
        const { backup, value } = item
        if (backup !== value) {
          this.backupAndEmitChange(item)
        }
      },
      onSelectChange(item) {
        this.backupAndEmitChange(item)
      },
      onRangeOpenChange(open, item) {
        item.visible = open
        const { momentEqual, backupAndEmitChange } = this
        const { value, backup, format } = item
        if (!open && !momentEqual(value, backup, format)) {
          backupAndEmitChange(item, [moment(value[0]), moment(value[1])])
        }
      },
      onRangeChange(dates, datesStr, item) {
        const { momentEqual, backupAndEmitChange } = this
        const { value, backup, format } = item
        if (!item.visible && !momentEqual(value, backup, format)) {
          backupAndEmitChange(item, [moment(value[0]), moment(value[1])])
        }
        item._t = dates[1]
      },
      onRangeCalendarChange(dates, item) {
        item._f = dates[0]
        item._t = null
      },
      disabledRangeDate(current, item) {
        const { maxDays, _f, _t } = item
        if (maxDays && _f && !_t) {
          const selectV = _f.valueOf()
          const microSeconds = maxDays * 1000 * 3600 * 24
          return (
            current > moment(new Date(selectV + microSeconds), 'YYYY-MM-DD') ||
            current < moment(new Date(selectV - microSeconds), 'YYYY-MM-DD')
          )
        } else {
          return false
        }
      },
      onCalendarOpenChange(open, item) {
        item.visible = open
        const { momentEqual, backupAndEmitChange } = this
        const { value, backup, format } = item
        if (!open && !momentEqual(value, backup, format)) {
          backupAndEmitChange(item, moment(value))
        }
      },
      onCalendarChange(date, dateStr, item) {
        const { momentEqual, backupAndEmitChange } = this
        const { value, backup, format } = item
        if (!item.visible && !momentEqual(value, backup, format)) {
          backupAndEmitChange(item, moment(value))
        }
      },
      onDateChange(item) {
        const { momentEqual, backupAndEmitChange } = this
        const { value, backup, format } = item
        if (!momentEqual(value, backup, format)) {
          backupAndEmitChange(item, moment(value))
        }
      },
      getFormat(item) {
        if (item && item.format) {
          return item.format
        }
        const type = item.type
        switch (type) {
          case 'time':
            return 'HH:mm:ss'
          case 'month':
            return 'YYYY-MM'
          case 'date':
          case 'range':
            return 'MM-DD HH'
          case 'datetime':
            return 'YYYY-MM-DD HH:mm:ss'
          default:
            return undefined
        }
      },
      backupAndEmitChange(item, backValue = item.value) {
        const { getFormValues, getFormOptions } = this
        item.backup = backValue
        this.formValues = getFormValues(this.formItems)
        this.formOptions = getFormOptions(this.formItems)
        this.$emit('change', this.formValues, this.formOptions)
      },
      getFormValues(items = this.formItems) {
        const formValues = {}
        items
          .filter(
            (item) =>
              item.value !== undefined && item.value !== '' && item.value !== null
          )
          .forEach((item) => {
            const { value, format, type, expandDayTime } = item
            if (format) {
              if (typeof format === 'function') {
                formValues[item.dataIndex] = format(item.value)
              } else if (typeof format === 'string' && type === 'range') {
                formValues[item.dataIndex] = value.map((e) => e.format(format))
                if (expandDayTime === true) {
                  const [d0, d1] = formValues[item.dataIndex]
                  formValues[item.dataIndex] = [
                    moment(d0).format('YYYY-MM-DD 00:00:00'),
                    moment(d1).format('YYYY-MM-DD 23:59:59'),
                  ]
                }
              } else if (
                typeof format === 'string' &&
                value.constructor.name === 'Moment'
              ) {
                formValues[item.dataIndex] = value.format(format)
              } else {
                formValues[item.dataIndex] = value
              }
            } else {
              formValues[item.dataIndex] = value
            }
          })
        return formValues
      },
      getFormOptions(items = this.formItems) {
        return items
          .filter(
            (item) =>
              item.value !== undefined && item.value !== '' && item.value !== null
          )
          .map(({ key, value, format }) => ({
            field: key,
            value: value,
            format: format,
          }))
      },
      onVisibleChange(item, index) {
        if (!item.visible) {
          item.value = item.backup
        } else {
          let input = document.getElementById(`${this.formIdPrefix}${index}`)
          if (input) {
            setTimeout(() => {
              input.focus()
            }, 0)
          } else {
            this.$nextTick(() => {
              input = document.getElementById(`${this.formIdPrefix}${index}`)
              input.focus()
            })
          }
        }
      },
      momentEqual(target, source, format) {
        if (target === source) {
          return true
        } else if (Array.isArray(target) && Array.isArray(source)) {
          if (target.length !== source.length) {
            return false
          }
          for (let i = 0; i < target.length; i++) {
            if (!this.momentEqual(target[i], source[i], format)) {
              return false
            }
          }
          return true
        } else if (
          target &&
          source &&
          target.format(format) === source.format(format)
        ) {
          return true
        }
        return false
      },
      mouseenter() {
        clearTimeout(this.mouseTimer)
        this.mouseon = true
      },
      mouseleave() {
        const vm = this
        this.mouseTimer = setTimeout(() => (vm.mouseon = false), 2000)
      },
    },
  }
</script>

<style scoped lang="less">
  .form-area-hover {
    &:hover {
      border: 1px solid @primary-color;
      border-radius: 4px;
      padding: 2px;
    }
  }

  .form-area {
    .select-root {
      text-align: left;
    }

    margin: -4px 0;

    .form-item {
      margin: 4px 4px;
      display: inline-block;

      .title {
        padding: 4px 8px;
        cursor: pointer;
        border-radius: 4px;
        user-select: none;
        display: inline-flex;
        align-items: center;

        .close {
          color: @text-color-second;
          margin-left: 4px;
          font-size: 12px;
          vertical-align: middle;

          :hover {
            color: @text-color;
          }
        }

        .range-picker {
          text-align: left;
          margin-left: 4px;
          width: 200px;
        }

        .datetime-picker {
          margin-left: 4px;
          width: 159px;
        }

        .date-picker {
          margin-left: 4px;
          width: 104px;
        }

        .time-picker {
          margin-left: 4px;
          width: 82px;
        }

        .switch {
          margin-left: 4px;
        }

        .value {
          display: inline-block;
          overflow: hidden;
          flex: 1;
          vertical-align: top;
          max-width: 120px;
          padding-bottom: '5px;';
          text-overflow: ellipsis;
          word-break: break-all;
          white-space: nowrap;
        }

        &.active {
          background-color: @layout-bg-color;
        }
      }

      .icon-down {
        vertical-align: middle;
        font-size: 12px;
      }
    }

    .form-overlay {
      padding: 8px 0px;
      text-align: center;
    }

    .select {
      margin-left: 4px;
      max-width: 154px;
      min-width: 94px;
      height: 24px;
      text-align: left;
      overflow: hidden;
    }

    .operations {
      display: flex;
      margin: -6px 0;
      justify-content: space-between;

      .btn {}
    }
  }
</style>