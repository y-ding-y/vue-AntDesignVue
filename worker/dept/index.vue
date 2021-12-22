<template>
	<ef-main :isMainPage="true" :loading="loading">

		<a-descriptions size="small" bordered v-for="(item,index) in deptinfo" :key="index">
			<a-descriptions-item :label="$t('plant_name')" :span='1'>
				<a-select v-if="item.isaction" v-model="item.plant" style="width: 300px;">
					<a-select-option v-for="(rec,i) in plantlist" :value="rec.id" :key="i" :title="rec.name">
						{{rec.name}}
					</a-select-option>
				</a-select>
				<a-input :value="item.plant_name" v-else disabled> </a-input>
			</a-descriptions-item>
			<a-descriptions-item :label="$t('dept')" :span='2'>
				<a-input v-model="item.dept_name" v-if="item.isaction" style="width: 300px;" disabled></a-input>
				<a-input v-model="item.dept_name" v-else style="width: 300px;" disabled></a-input>
			</a-descriptions-item>
			<a-descriptions-item :label="$t('files')" :span='3'>
				<div v-if="!item.isaction">
					<transition v-if="item.layout">
						<img :src="icons+'/file/'+item.layout+'/small' " :large="icons+'/file/'+item.layout+'/large'"
							class="img" :preview="item">
					</transition>
				</div>
				<div v-else>
					<a-upload :action="fileapi + '/file/upload'" @change="handleChange" list-type="picture"
						:file-list="fileList">
						<a-button type="primary">
							<a-icon type="upload" /> Upload
						</a-button>
					</a-upload>
				</div>
			</a-descriptions-item>
			<a-descriptions-item :label="$t('actions')">
				<div v-if="item.isaction">
					<a-button v-if="actionstatus=='update'" :class="'success'" @click="handle_saveupdate(item)">
						<a-icon type="form" />Save Update
					</a-button>
					&nbsp;&nbsp;&nbsp;
					<a-button v-if="actionstatus=='add'" type="primary" ghost @click="submit(item,index)">
						<a-icon type="save" />Add
					</a-button>
					<a-button type="primary" ghost @click="handle_close(item,index)">
						<a-icon type="close" />Close
					</a-button>
				</div>
				<div v-else>
					<a-button :class="'success'" @click="handle_update(item,index)">
						<a-icon type="form" />Update
					</a-button>
					&nbsp;&nbsp;&nbsp;
					<a-popconfirm title="Are you sure delete ?" ok-text="Yes" cancel-text="No" @confirm="handle_delete">
						<a-button type="primary" ghost>
							<a-icon type="delete" />Delete
						</a-button>
					</a-popconfirm>
				</div>
			</a-descriptions-item>
		</a-descriptions>
	</ef-main>

</template>
<script>
	import { reqPost } from '@/services/base'
	import { merge } from '@/utils/util'
	import EfMain from '@/components/page/main'


	const i18n = require('./i18n')
	export default {
		name: 'workerDept',
		components: { EfMain },
		props: {
			dept: { type: String, default: "" },
		},
		i18n: merge(require('@/i18n'), i18n),
		data() {
			return {
				loadcount: 0,
				i: 0,
				deptlist: [],
				deptinfo: [],
				fileList: [],
				isactive: "1",
				deptid: "",
				isedit: false,
				updateitems: [],
				layoutid: "",
				temp_dept: {},
				actionstatus: "",
				plantlist: [],
			};
		},

		computed: { 
			icons() {
				return process.env.VUE_APP_API_WEB_URL;
			},
			fileapi() {
				return process.env.VUE_APP_API_BASE_URL
			},
			loading() {
				return this.loadcount > 0;
			},
		},
		created() {
			this.loadcount = 1;
			reqPost('PARA_SET', { paras: [{ "key": "PARA.163" }] }).then(res => {
				this.loadcount--;
				if (res.data.length > 0) {
					this.plantlist = res.data[0].data;
				}
				else {
					this.$message.error("請先維護部門")
				}
			})

			this.loadcount = 1;
			reqPost('WORKER', { prop: "getdeptid", name: this.$props.dept }).then(res => {
				this.loadcount--;
				if (res.data != null) {
					this.deptid = res.data;
					this.handle_search();
					this.actionstatus = "update"
				}
				else {
					var temp = { dept_name: this.$props.dept, isaction: true, plant_name: "", plant: "" };
					this.deptinfo.splice(0, 1, temp)
					this.actionstatus = "add";
				}
			})
		},
		methods: {
			handleChange(info) {
				info.fileList.map(file => {
					if (file.response) {
						this.layoutid = file.response.id
					}
				});
				this.fileList = info.fileList;
			},
			handle_search() {
				this.loadcount = 1;
				reqPost('WORKER', { prop: "getdept", id: this.deptid }).then(res => {
					this.loadcount--;
					if (res.data.length > 0) {
						this.deptinfo = res.data.map(e => {
							e.isaction = false
							return e
						});
					}
				})
			},
			submit(values) {
				if (this.fileList.length <= 0) {
					this.$message.error("請選擇附件上傳")
				}
				else if (this.fileList.length > 1) {
					this.$message.error("Only one picture")
				}
				else if (this.fileList[0].type.split('/')[0] != "image") {
					this.$message.error("Only upload Image")
				}
				else {
					values.fileuid = this.fileList[0].response.id;
					if (values.plant != "") {
						values.prop = "insertdept";
						values.name = this.$props.dept
						this.loadcount = 1;
						reqPost("WORKER", values).then(res => {
							if (res.data) {
								if (res.data.type == "ok") {
									this.$message.success("Add Success")
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
				reqPost("WORKER", { prop: "deletedept", id: this.deptid }).then(res => {
					if (res.data > 0) {
						this.i++;
						this.deptid = "";
						this.$message.success("Delete Success")
						this.deptinfo = [{ plant_name: "", layout: "", dept_name: this.$props.dept, isaction: true }]
					}
					else {
						this.$message.error("Delete ERROR")
					}
				})
			},

			handle_update(e, index) {
				e.isaction = true;
				this.actionstatus = "update";
				e.plant = Number(e.plant)
				this.temp_dept = { ...e };
				this.deptinfo.splice(index, 1, e);
			},
			handle_saveupdate(values) {
				if (this.fileList.length <= 0) {
					this.$message.error("請選擇附件上傳")
				}
				else if (this.fileList.length > 1) {
					this.$message.error("Only one picture")
				}
				else if (this.fileList[0].type.split('/')[0] != "image") {
					this.$message.error("Only upload Image")
				}
				else {
					values.fileuid = this.fileList[0].response.id
					if (values.plant != "") {
						values.prop = "updatedept";
						values.id = this.deptid;
						values.name = values.dept_name
						this.loadcount = 1;
						reqPost("WORKER", values).then(res => {
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

			handle_close(e, index) {
				if (this.actionstatus == "update") {
					var temp = { ...this.temp_dept }
					temp.isaction = false;
					this.deptinfo.splice(index, 1, temp);
				}
				else if (this.actionstatus == "add") {
					this.fileList = [];
					var temp1 = { dept_name: this.$props.dept, isaction: true, plant_name: "", plant: "" };
					this.deptinfo.splice(0, 1, temp1)
				}
			},
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
</style>