<template>
	<div>
		<a-modal :title="$t('dialog_title')" :visible="visible" :width="listWidth" :confirm-loading="confirmLoading"
			@cancel="handleCancel" :footer="null">
			<div style="display: flex;justify-content: space-between;">
				<a-checkbox-group v-model="value" name="checkboxgroup" @change="onChange" :key="key">
					<a-checkbox v-for="item in types" :value="item.key" :key="item.key">{{
						item.value
						}}</a-checkbox>
				</a-checkbox-group>
				<div class="actions">
					<slot name="actions"></slot>
					<a-tooltip :title="$t('exportData')">
						<a-icon @click="onexportData" type="file-excel" />
					</a-tooltip>
				</div>
			</div>

			<a-table :size="size" :data-source="tabledata" :columns="columns" bordered :pagination="false"
				@change="onTableChange" :rowKey="(record) => record.rowno" :loading="loading"
				:scroll="{ y: pageMinHeight - 160, x: listTableWidth  }" id="dataTableId">
				<template slot="no" slot-scope="t, r, i">{{ i + 1 }}</template>
				<span slot="rc">
					{{ rcname }}
				</span>
			</a-table>
		</a-modal>
	</div>
</template>

<script>
	import { mapState } from 'vuex'
	import { reqPost } from '@/services/base'
	import bus from './bus.js'
	import { merge } from '@/utils/util'
	import { saveToExcel } from '@/utils/util'

	export default {
		name: "g5io_dialog",
		i18n: merge(require('@/i18n'), require('./i18n')),
		props: {
			actionFix: String,
			vshow: Boolean,
			params: Object,
			rc: String,
			rcname: String,
			size: String,
			itemkey: String,
			group: String,
		},
		data() {
			return {
				m_chose: '',
				dataTableId: '',
				visible: false,
				confirmLoading: false,
				loading: false,
				tabledata: [],
				sortOrder: '',
				columns: [],
				value: [],
				key: 0,
				listWidth:
					document.documentElement.clientWidth * 0.8 > 2000
						? 2000
						: document.documentElement.clientWidth * 0.8,
				listTableWidth: 2100,
			}
		},
		computed: {
			...mapState('setting', ['pageMinHeight']),

			types() {
				return [
					{ key: 'familyName', value: this.$t('family') },
					{ key: 'pn', value: this.$t('pn') },
					{ key: 'pnDesc', value: this.$t('pndesc') },
					{ key: 'wo', value: this.$t('wo') },
					{ key: 'dateTime', value: this.$t('date') },
					{ key: 'month', value: this.$t('month') },
					{ key: 'week', value: this.$t('week') },
					{ key: 'dt', value: this.$t('day') },
				]
			},
		},
		watch: {
			vshow() {
				if (this.vshow) {
					this.visible = true
					this.m_chose = 'all'
					this.value = [];
					this.handle_get()
				}
			},
		},
		methods: {
			handleCancel() {
				this.visible = false
				bus.$emit('aavshow', false)
			},

			handle_get() {
				this.params.routeCode = String(this.rc)
				this.params.prop = "io";
				this.params[this.group] = this.itemkey
				//    this.params.group = "targetQty,familyName,week,routeCode,pn,lineName,dateTime,wo";
				this.params.group = [
					'targetQty',
					'familyName',
					'week',
					'routeCode',
					'pn',
					'lineName',
					'dateTime',
					'wo',
					"pnDesc",
					"class",
					"type"
				]
				//this.params.select = "targetQty,familyName as NAME,wo,dateTime AS DATA,lineName,pn,week as KEY,routeCode,sum(qty) AS QTY,lineOutputQty AS OUTPUT";
				this.params.select = {
					"targetQty": 'targetQty',
					"FAMILYNAME": 'familyName',
					"wo": 'wo',
					"DATA": 'dateTime',
					"lineName": 'lineName',
					"pn": 'pn',
					"KEY": 'week',
					"routeCode": 'routeCode',
					"QTY": 'sum(qty) ',
					"OUTPUT": 'lineOutputQty',
					"PNDESC": "pnDesc",
					"CLASS": "class",
					"TYPE": "type"
				}
				this.params.order = ''
				this.params.type = 'O'
				this.loading = true

				this.columns = [
					{ title: this.$t('no'), scopedSlots: { customRender: 'no' }, key: 'no', width: 80, align: 'center', },
					{ title: this.$t('class'), dataIndex: 'class', key: 'class', width: 120, },
					{ title: this.$t('family'), dataIndex: 'familyname', key: 'familyname', width: 110, },
					{ title: this.$t('line'), dataIndex: 'linename', key: 'linename', width: 120, },
					{ title: this.$t('routeCode'), dataIndex: 'route_code', width: 130, key: 'route_code', scopedSlots: { customRender: 'rc' }, },
					{ title: this.$t('pn'), dataIndex: 'pn', key: 'pn', width: 140 },
					{ title: this.$t('pndesc'), dataIndex: 'pndesc', key: 'pndesc', },
					{ title: this.$t('wo'), dataIndex: 'wo', key: 'wo', width: 140 },
					{ title: this.$t('date'), dataIndex: 'data', key: 'data', width: 180 },
					{ title: this.$t('input'), dataIndex: 'qty', key: 'qty', align: 'right', width: 90, },
					{ title: this.$t('type'), dataIndex: 'type', key: 'type', width: 90, },
					{ title: this.$t('target'), dataIndex: 'targetqty', key: 'targetqty', align: 'right', width: 90 },
				].map((e) => {
					e.sorter =
						e.key === 'rowno' ? false : (a, b) => this.sorter(a, b, e.key)
					return e
				})
				reqPost('SFCS_SMT', this.params)
					.then((res) => {
						this.loading = false
						this.tabledata = res.data.map((e) => {
							e.qty = Number(e.qty)
							e.targetqty = Number(e.targetqty)
							return e
						})
						return res.data
					})
					.catch((error) => {
						console.log(error)
					})
			},

			get_value(item, key) {
				for (var i = 0; i < item.length; i++) {
					if (item[i].key === key) {
						return item[i].value
					}
				}
			},

			onexportData() {
				saveToExcel('dataTableId')
			},

			sorter(a, b, rcs) {
				const aN = a[rcs]
				const bN = b[rcs]
				return aN === bN ? 0 : aN > bN ? 1 : -1
			},

			onTableChange(pagination, filters, sorter) {
				this.sortOrder = sorter.order
			},

			onChange() {
				if (this.value.length == 0) {
					this.m_chose = 'all'
					this.handle_get()
				} else {
					this.params.prop = "io";
					this.params.routeCode = String(this.rc)
					// this.params.group = this.m_chose;
					this.params[this.group] = this.itemkey
					this.params.group = this.value
					this.params.type = 'O'
					var temp = {}
					var arr = []
					for (var i = 0; i < this.value.length; i++) {
						temp[this.value[i].toUpperCase()] = this.value[i]
						arr.push({
							title: this.get_value(this.types, this.value[i]),
							dataIndex: this.value[i].toLowerCase(),
							key: this.value[i].toLowerCase(),
							align: 'center',
						})
					}
					temp.QTY = 'sum(qty)'
					this.params.select = temp

					this.columns = [
						{ title: this.$t('no'), dataIndex: 'rowno', key: 'rowno', width: 80 },
						...arr,
						{
							title: this.$t('input'),
							dataIndex: 'qty',
							key: 'qty',
							align: 'right',
						},
					].map((e) => {
						e.sorter =
							e.key === 'rowno' ? false : (a, b) => this.sorter(a, b, e.key)
						return e
					})
					this.loading = true
					reqPost('SFCS_SMT', this.params)
						.then((res) => {
							this.loading = false
							this.tabledata = res.data.map((e) => {
								e.qty = Number(e.qty)
								return e
							})
							return res.data
						})
						.catch((error) => {
							console.log(error)
						})
				}
			},
		},
	}
</script>

<style scoped lang="less">
	.actions {
		margin-top: 4px;
		text-align: right;
		font-size: 17px;
		color: @text-color;

		* {
			margin: 0 4px;

			&:hover {
				color: @primary-color;
			}
		}
	}
</style>