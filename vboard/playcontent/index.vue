<template>
	<ef-main :isMainPage="true" :loading="loading" :key="formi" :goback="true">
		<a-descriptions bordered>
			<a-descriptions-item label="頻道名稱">
				<b>{{channelname}}</b>
			</a-descriptions-item>
			<a-descriptions-item label="節目單名稱">
				<b>{{name}}</b>
			</a-descriptions-item>
			<a-descriptions-item label="播放時間">
				<b> {{time}}</b>
			</a-descriptions-item>
		</a-descriptions>
		<br>

		<a-row :gutter="20">
			<a-col :span="8">

				<a-form-model>
					<a-form-model-item>
						<div style="width: 100%;border: 1px solid  #d9d9d9;border-radius: 4px;">
							<span style="padding: 24px;"><b>待選列表</b></span>
							<div :style="demoInfiniteContainer">
								<a-spin :spinning="sourceloading" />
								<div v-for="(item,index) in source" :key="index" class="target">
									{{item.name}}
									<div style="height: 100%; ">
										<a-button :class="'success'" size="small" icon="plus"
											@click="handle_chose(item,index)">
										</a-button>
									</div>
								</div>
							</div>

						</div>
					</a-form-model-item>
				</a-form-model>
				<!-- <div :style="demoInfiniteContainer">
					<a-spin :spinning="sourceloading" />
					<a-list bordered :data-source="source">
						<div slot="header">
							<b>全部視頻</b>
						</div>

						<a-list-item slot="renderItem" slot-scope="item">
							<div style="width: 100%; display: flex;justify-content: space-between;">
								{{ item.name }}
								<a-button :class="'success'" size="small" icon="plus" @click="handle_chose(item)">
								</a-button>
							</div>
						</a-list-item>
					</a-list>
				</div> -->
			</a-col>
			<a-col :span="16">
				<a-form-model>
					<a-form-model-item>
						<div style="width: 100%;border: 1px solid  #d9d9d9;border-radius: 4px;">
							<span style="padding: 24px;"><b>節目單視頻列表</b></span>
							<div :style="demoInfiniteContainer">
								<a-spin :spinning="targetloading" />
								<div v-dragging="{  item: item, list: target,group:'target'}"
									v-for="(item,index) in target" :key="index" class="target">
									{{index+1}}- {{item.name}}
									<div style="height: 100%; ">
										<a-button :class="'warning'" size="small" icon="minus"
											@click="handle_delete(item,index)">
										</a-button>
									</div>
								</div>
							</div>

						</div>
					</a-form-model-item>
					<a-button type="primary" style="width: 100%;" @click="handle_addgroup">提交
					</a-button>
				</a-form-model>
			</a-col>
		</a-row>

	</ef-main>
</template>
<script>
	import { reqPost } from '@/services/base'
	import { merge } from '@/utils/util'
	import EfMain from '@/components/page/main'
	import { mapState } from 'vuex'


	const rules = {
		name: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
		timefrom: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
		timeto: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
		channel: [{ required: true, message: 'Cannot be empty', trigger: 'blur' },],
	}

	const form = {
		timefrom: ['12', '00'],
		timeto: ['12', '00'],
		name: "",
		channel: "",
	}


	const i18n = require('./i18n')
	export default {
		name: 'vboardplaybill',
		components: { EfMain, },
		i18n: merge(require('@/i18n'), i18n),
		props: {
			dept: { type: String, default: "" },
		},
		data() {
			return {
				tableId: 'vboardchannel' + new Date().format('hiu'),
				loadcount: 0,
				sourcecount: 0,
				targetcount: 0,
				source: [],
				target: [],

				actionstatus: "",
				id: 0,
				rules, form,
				visible: false,
				axiosprop: { "add": "insertplaybill", "update": "updateplaybill" },
				channels: [],
				formi: 0,
				i: 0,
				mockData: [],
				targetKeys: [],

				channel: "",
				playbill: "",

				channelname: "",
				time: "",
				name: "",
			};
		},
		watch: {
			dept() {
				this.id++;
			},
		},
		computed: {
			formItems() {
				var arr = [
					{ key: 'channel', title: this.$t('channel'), type: 'select', options: this.channels },
					{ key: 'name', title: this.$t('name'), type: 'input' },
					{ key: 'timefrom', title: this.$t('timefrom'), type: 'selecttime', options: this.options },
					{ key: 'timeto', title: this.$t('timeto'), type: 'selecttime', options: this.options },
				]
				return arr
			},
			icons() {
				return process.env.VUE_APP_API_WEB_URL;
			},
			dialogwidth() {
				return document.documentElement.clientWidth / 2;
			},
			...mapState('setting', ['lang', 'pageMinHeight']),
			scroll() {
				return { x: 1100, y: this.pageMinHeight - 200 }
			},
			loading() {
				return this.loadcount > 0;
			},
			sourceloading() {
				return this.sourcecount > 0
			},
			targetloading() {
				return this.targetcount > 0
			},
			demoInfiniteContainer() {
				return {
					' border': '1px solid #e8e8e8',
					'borderRadius': '4px',
					'overflowY': 'scroll',
					'padding': '8px 24px',
					'height': Number(this.pageMinHeight - 265) + 'px',
				}
			},
		},
		created() {
			if (Object.values(this.$route.query).length > 0) {
				this.channelname = this.$route.query.channelname;
				this.name = this.$route.query.name;
				this.time = this.$route.query.time;

				this.channel = this.$route.query.channel;
				this.playbill = this.$route.query.playbill;
				this.sourcecount = 1
				reqPost('VBOARD', { prop: "getcontent", channel: this.channel }).then((res) => {
					this.sourcecount--;
					this.source = res.data;
				})
				this.targetcount = 1;
				reqPost('VBOARD', { prop: "getplaylist", playbill: this.playbill }).then((res) => {
					this.targetcount--
					this.target = res.data;
				})

			}
			else {
				//
			}
		},
		methods: {
			handle_addgroup() {
				var item = [];
				item = this.target.map((e, i) => {
					return {
						playbill: this.playbill,
						content: e.id,
						seq: i + 1
					}
				})
				var params = {
					prop: "insertplaylist",
					playbill: this.playbill,
					items: item
				}
				this.loadcount = 1;
				reqPost('VBOARD', params).then((res) => {
					if (res.data) {
						this.loadcount--;
						this.$message.success("Success")
					}
					else {
						this.$message.error("Error")
					}
				})
			},
			handle_chose(e) {
				this.i++;
				var temp = { ...e }
				temp.key = this.i;
				this.target.push(temp)
			},
			handle_delete(e, index) {

				this.target.splice(index, 1)
			},
		},
		mounted() {
		},

	};
</script>


<style scoped lang="less">
	.title {
		font-weight: bold;
	}

	.basebg {
		background-color: @base-bg-color;
	}

	.success,
	.success:hover,
	.success:visited {
		/* background-color: @success-color; */
		color: @success-color;
	}

	.warning,
	.warning:hover,
	.warning:visited {
		/* background-color: @success-color; */
		color: @warning-color;
	}

	.img {
		width: 2rem;
		height: 2rem;
		border-radius: .04rem;
		margin-left: .3rem;
		margin-top: .3rem;
		object-fit: cover;
	}

	.target {
		border-top: 1px solid #f0f0f0;
		padding-right: 24px;
		padding-left: 24px;
		display: flex;
		justify-content: space-between;
	}

	*::-webkit-scrollbar {
		width: 7px;
		height: 10px;
		background-color: transparent;
	}

	/*定义滚动条高宽及背景 高宽分别对应横竖滚动条的尺寸*/
	*::-webkit-scrollbar-track {
		background-color: @primary-4;
	}

	/*定义滚动条轨道 内阴影+圆角*/
	*::-webkit-scrollbar-thumb {
		background-color: @primary-color;
		border-radius: 6px;
	}

	/*定义滑块 内阴影+圆角*/
	.scrollbarHide::-webkit-scrollbar {
		display: none
	}

	.scrollbarShow::-webkit-scrollbar {
		display: block
	}
</style>