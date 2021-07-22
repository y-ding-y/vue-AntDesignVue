<template>
  <div class="rank" style="height:{height};">
    <h4 class="title" v-if="title !== null && title !== ''">{{ title }}</h4>
    <ul class="list">
      <li style="margin-top:0px;">
        <span style="background:none;"></span>
        <span v-for="col in columns" :key="col.key" :style="col.style">{{
          col.name
        }}</span>
      </li>
      <li :key="index" v-for="(item, index) in list">
        <span :class="index < 3 ? 'active' : null">{{ index + 1 }}</span>
        <span v-for="col in columns" :key="col.key" :style="col.style">{{
          item[col.key]
        }}</span>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'RankingList',
  props: {
    height: { default: '100ppx' },
    title: { defualt: '' },
    list: { default: [] },
    columns: {
      type: Array,
      default: () => [
        { key: 'name', style: {} },
        { key: 'total', style: {} },
      ],
    },
  },
}
</script>

<style lang="less" scoped>
.rank {
  padding: 0;
  .title {
  }
  .list {
    margin: 10px 0 0;
    padding: 0;
    list-style: none;
    li {
      margin-top: 16px;
      span {
        color: @text-color-second;
        font-size: 14px;
        line-height: 22px;
        display: inline-block;
      }
      span:first-child {
        background-color: @layout-bg-color;
        border-radius: 20px;
        display: inline-block;
        font-size: 12px;
        font-weight: 600;
        margin-right: 24px;
        height: 20px;
        line-height: 20px;
        width: 20px;
        text-align: center;
      }
      span.active {
        background-color: #314659 !important;
        color: @text-color-inverse !important;
      }
      span:last-child {
        float: right;
      }
    }
  }
}
</style>
