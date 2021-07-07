<template>
	<div>
		<ef-main :isMainPage="true" :title="$t('etdtitle')" :loading="loading" @resize="resize">

			<a-steps :current="active" finish-status="finish" align-center key=key1>
				<a-step>
					<template slot="description">
						<br>
						<a-upload :file-list="fileList" :remove="handleRemove" :before-upload="beforeUpload"
							@change="setfile">
							<a-button type="primary">
								<a-icon type="upload" /> Select File
							</a-button>
						</a-upload>
					</template>
				</a-step>
				<a-step>
					<template slot="description">
						<br>
						<a-button type="primary" @click="uploadexcel">Click Upload <i class="a-icon-upload"></i>
						</a-button>
					</template>
				</a-step>
				<a-step>
					<template slot="description">
						<br>
						Success
					</template>
				</a-step>
			</a-steps>
			<br>

			<a-table class="basebg" :scroll="{  y: tableheight }" :bordered="true" :columns="columns"
				:pagination="false" :data-source="tabledata" rowKey="NO" :size="sSize"
				:row-selection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }">
				<template slot="NO" slot-scope="txt, rec,i">
					{{i+1}}
				</template>

			</a-table>

		</ef-main>

	</div>
</template>

<script>
	import { reqPost } from '@/services/base'
	import { readExcel } from './upload.js'
	import moment from 'moment'
	import EfMain from '@/components/page/main'
	import { merge } from '@/utils/util'
	import { mapState } from 'vuex'

	const i18n = require('./i18n')
	export default {
		name: 'efetd',
		i18n: merge(require('@/i18n'), i18n),
		data() {
			return {
				fileList: [],
				active: 0,
				filedata: "",
				tabledata: [],
				total: 0,
				pagesize: 10,
				currentPage: 1,
				key1: 1,
				selectedRowKeys: [],
				sSize: 'small',
			};
		},
		components: { EfMain },
		computed: {

			loading() {
				return this.loadcount > 0
			},
			...mapState('setting', ['pageMinHeight']),
			tableheight() {
				return this.pageMinHeight - 330;
			},
			columns() {
				var temp = [{ key: "NO", width: 100 }];
				var i = 1;

				for (var key in this.tabledata[0]) {
					if ([10, 13, 16, 19, 22].includes(i)) {
						temp.push({ key: key })
					}
					i++;
				}

				return temp.map(e => {
					e.dataIndex = e.key
					e.title = e.dataIndex
					e.slots = { customRender: e.key }
					e.scopedSlots = e.scopedSlots ?? { customRender: e.slot ?? e.key }
					return e;
				})
			},
		},
		methods: {
			onSelectChange(selectedRowKeys) {
				this.selectedRowKeys = selectedRowKeys;
			},

			beforeUpload(file) {
				this.fileList = [file];
				return false;
			},
			getIndex($index) {  //表格序号 
				return ((this.currentPage - 1) * this.pagesize + $index + 1)
			},
			resize(size) {
				this.sSize = size
			},
			dateFormat(date) {
				return moment(date).format('YYYY-MM-DD')
			},
			clear() {
				++this.key1;
				this.handleRemove();
			},
			handleRemove() {
				this.fileList.length = 0;
				this.tabledata.length = 0;
				this.selectedRowKeys = [];
				this.active = 0;
			},
			uploadexcel() {
				var temp = this.selectedRowKeys.map(i => {
					return {
						mtcSo: this.tabledata[i][this.columns[1].key],
						wkDate: this.tabledata[i][this.columns[2].key],
						confirmation: this.tabledata[i][this.columns[3].key],
						soLineId: this.tabledata[i][this.columns[4].key],
						deliveryDetailId: this.tabledata[i][this.columns[5].key],
					}
				})
				if (this.selectedRowKeys.length > 0) {
					this.active = this.active + 1;
					var params = {
						prop: 'insertEtd',
						site: 'GTK',
						rows: temp,
					}
					this.loadcount = 1
					reqPost('EF_PC_INSERTETD', params).then((res) => {
						this.loadcount--
						if (res.data) {
							this.$message.success('Success')
							this.clear()
						} else {
							this.$message.error('Fail')
						}
					})
				}
				else {
					this.$message.error('至少选择一条数据')
				}
			},
			setfile(e) {
				this.filedata = e.file;
				var rr = [];
				this.tabledata.length = 0;
				this.total = 0;
				rr = readExcel(e.file)
				this.loadcount = 1
				setTimeout(() => {
					this.loadcount--;
					this.tabledata = rr[0].sheet.map((e, i) => {
						e.NO = i
						e[' ETD '] = this.dateFormat(e[' ETD '])
						return e;
					});
				}, 1000);  //1秒后执行代码
				this.active = this.active + 1;
			},
		}
	}
</script>

<style>
	.a-upload {
		width: 120px;
		height: 35px
	}
</style>
