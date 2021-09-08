<template>
	<ef-main :isMainPage="true" :loading="loading" :openSubmit="true">

		<a-tabs v-model="isactive">
			<a-tab-pane key="1" tab="新增">
				<upload-form :isMainPage="false" :istitile="false" :form-items="formItems" @submit="submit"
					:loading="loading">
				</upload-form>
			</a-tab-pane>
			<a-tab-pane key="2" tab="查询" force-render>
				<ef-main :isMainPage="false" :title="  ' '" :loading="loading" :form-items="searchItems"
					@submit="handle_get" :key="i">
					<div v-if="!isedit">
						<a-descriptions size="small" bordered v-for="(item,index) in deptinfo" :key="index">
							<a-descriptions-item :label="$t('plant_name')" :span='1'>
								<a-input :value="item.plant_name" disabled> </a-input>
							</a-descriptions-item>
							<a-descriptions-item :label="$t('dept')" :span='2'>
								<a-input v-model="item.dept_name" style="width: 300px;" disabled>12</a-input>
							</a-descriptions-item>
							<a-descriptions-item :label="$t('files')" :span='3'>
								<transition v-if="item.layout">
									<img :src="icons+'/file/'+item.layout+'/small' "
										:large="icons+'/file/'+item.layout+'/large'" class="img" :preview="item">
								</transition>
							</a-descriptions-item>
							<a-descriptions-item :label="$t('actions')" v-if="deptid">
								<div>
									<a-button type="primary" ghost @click="handle_update">
										<a-icon type="form" />Update
									</a-button>

									<a-popconfirm title="Are you sure delete ?" ok-text="Yes" cancel-text="No"
										@confirm="handle_delete">
										<a-button type="primary" ghost>
											<a-icon type="delete" />Delete
										</a-button>
									</a-popconfirm>
								</div>
							</a-descriptions-item>
						</a-descriptions>
					</div>

					<upload-form v-if="isedit" :isMainPage="false" :istitile="false" :form-items="updateitems"
						@submit="handle_saveupdate" :loading="loading">
					</upload-form>
				</ef-main>

			</a-tab-pane>

		</a-tabs>


	</ef-main>

</template>
<script>
	import { reqPost } from '@/services/base'
	import UploadForm from '@/components/page/uploadform'
	import { merge } from '@/utils/util'
	import EfMain from '@/components/page/main'
	const formItems = [
		{ key: 'plant', type: 'select', paraKey: 'para.163', allowClear: false, value: "165", span: "3" },
		{ key: 'name', type: 'input', span: "3", style: "width:300px;max-width:600px", },
		{ key: 'files', type: 'uploadfile', picture: true, api: process.env.VUE_APP_API_BASE_URL + "/file/upload", span: "3" },
	]
	const searchItems = [
		{ key: 'dept', type: 'select', paraKey: "WORKER.dept", allowClear: false, },
	]


	const i18n = require('./i18n')
	export default {
		name: 'oqaworkerdept',
		components: { UploadForm, EfMain },
		i18n: merge(require('./i18n'), i18n),
		data() {
			return {
				loadcount: 0,
				i: 0,
				deptlist: [],
				deptinfo: [{ plant_name: "", layout: "", dept_name: "" }],
				searchItems,
				formItems,
				isactive: "1",
				deptid: "",
				isedit: false,
				updateitems: [],
			};
		},
		watch: {

		},
		computed: {
			icons() {
				return process.env.VUE_APP_API_WEB_URL;
			},

			loading() {
				return this.loadcount > 0;
			},
		},
		created() {
			//this.loadcount = 1;
			// reqPost('WORKER_OQA', { prop: "getdept" }).then(res => {
			//     this.loadcount--;
			//     this.deptlist = res.data.map(e => {
			//         return {
			//             name: e.dept_name,
			//             key: e.id,
			//             title: e.dept_name
			//         }
			//     });
			// })
		},
		methods: {
			handle_get(values) {
				this.deptid = values.dept;
				this.handle_search();

			},
			handle_search() {
				this.loadcount = 1;
				this.isedit = false;
				reqPost('WORKER_OQA', { prop: "getdept", id: this.deptid }).then(res => {
					this.loadcount--;
					this.deptinfo = res.data;
				})
			},
			submit(values) {
				if (!values.files) {
					this.$message.error("請選擇附件上傳")
				}
				else if (values.files.length > 1) {
					this.$message.error("Only one picture")
				}
				else if (values.files[0].type.split('/')[0] != "image") {
					this.$message.error("Only upload Image")
				}
				else {
					values.fileuid = values.files[0].response.id
					console.log(values)
					if (values.name && values.fileuid) {
						values.prop = "insertdept";
						this.loadcount = 1;
						console.log(values)
						reqPost("WORKER_OQA", values).then(res => {
							if (res.data) {
								if (res.data.type == "ok") {
									this.$message.success("Upload Success")
								}
								else {
									this.$message.warning("Already Exist")

								}
								this.loadcount--;
								this.isactive = "2";
								this.i++;
								this.deptid = res.data.data;
								this.handle_search();
							}
							else {
								this.$message.error("Upload ERROR")
							}
						})
					}
					else {
						this.$message.warning("Some Items is empty")
					}
				}

			},

			handle_delete() {
				console.log(this.deptid)
				reqPost("WORKER_OQA", { prop: "deletedept", id: this.deptid }).then(res => {
					if (res.data > 0) {
						this.i++;
						this.deptid = "";
						this.searchItems[0].value = "";
						this.$message.success("Delete Success")
						this.deptinfo = [{ plant_name: "", layout: "", dept_name: "" }]
					}
					else {
						this.$message.error("Delete ERROR")
					}
				})
			},

			handle_update() {
				this.isedit = true;
				console.log(this.deptinfo)
				this.updateitems = [{ key: 'plant', value: this.deptinfo[0].plant, type: 'select', paraKey: 'para.163', allowClear: false, span: "3" },
				{ key: 'name', value: this.deptinfo[0].dept_name, type: 'input', span: "3", style: "width:300px;max-width:600px", },
				{ key: 'files', type: 'uploadfile', picture: true, api: process.env.VUE_APP_API_BASE_URL + "/file/upload", span: "3" },
				]
			},
			handle_saveupdate(values) {
				if (!values.files) {
					this.$message.error("請選擇附件上傳")
				}
				else if (values.files.length > 1) {
					this.$message.error("Only one picture")
				}
				else if (values.files[0].type.split('/')[0] != "image") {
					this.$message.error("Only upload Image")
				}
				else {
					values.fileuid = values.files[0].response.id
					console.log(values)
					if (values.name && values.fileuid) {
						values.prop = "updatedept";
						values.id = this.deptid;
						this.loadcount = 1;
						console.log(values)
						reqPost("WORKER_OQA", values).then(res => {
							if (res.data) {
								this.$message.success("Update Success")
								this.loadcount--;
								this.isactive = "2";
								this.handle_search();
							}
							else {
								this.$message.error("Update ERROR")
							}
						})
					}
					else {
						this.$message.warning("Some Items is empty")
					}
				}
			},
		},
	};
</script>