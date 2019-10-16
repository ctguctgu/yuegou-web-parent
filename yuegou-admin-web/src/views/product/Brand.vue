<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="关键字"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getbrands">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="brands" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="60">
			</el-table-column>
			<el-table-column prop="name" label="商品名称" width="120" sortable>
			</el-table-column>
			<el-table-column prop="englishName" label="英文名" width="120" sortable>
			</el-table-column>
			<el-table-column prop="firstLetter" label="首字母" width="80" sortable>
			</el-table-column>
			<el-table-column prop="productType.name" label="商品类型" width="150" sortable>
			</el-table-column>
			<el-table-column prop="logo" label="LOGO" width="120">
				<template scope="scope">
					<img :src= "'http://172.16.4.153'+scope.row.logo" style="height: 50px;">
				</template>
			</el-table-column>
			<el-table-column prop="description" label="描述" min-width="150" sortable>
			</el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="size" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog title="编辑" v-model="editFormVisible" :close-on-click-modal="false">
			<el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
				<el-form-item label="商品名称" prop="name">
					<el-input v-model="editForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="英文名字" prop="englishName">
					<el-input v-model="editForm.englishName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品类型" prop="selectedOptions">
					<el-cascader
							:show-all-levels="false"
							expand-trigger="hover"
							:change-on-select="true"
							:props="defaultParams"
							:options="options"
							v-model="editForm.selectedOptions"
							:clearable="true"
					></el-cascader>
				</el-form-item>
				<el-form-item label="LOGO">
					<el-upload
							action="http://127.0.0.1:9999/services/common/file"
							list-type="picture-card"
							:on-preview="handlePictureCardPreview"
							:on-remove="handleRemove"
							:on-success="handleSuccess"
							:before-upload="handleBeforeUpload"
							:file-list="fileList"
					>
						<i class="el-icon-plus"></i>
					</el-upload>
					<el-dialog v-model="dialogVisible" size="tiny">
						<img width="100%" :src="dialogImageUrl" alt="">
					</el-dialog>
				</el-form-item>
				<el-form-item label="描述">
					<el-input type="textarea" v-model="editForm.description"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="editFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--新增界面-->
		<el-dialog title="新增" v-model="addFormVisible" :close-on-click-modal="false">
			<el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
				<el-form-item label="商品名称" prop="name">
					<el-input v-model="addForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="英文名字" prop="englishName">
					<el-input v-model="addForm.englishName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品类型" prop="selectedOptions">
					<el-cascader
							:show-all-levels="false"
							expand-trigger="hover"
							:change-on-select="true"
							:props="defaultParams"
							:options="options"
							v-model="addForm.selectedOptions"
							:clearable="true"
					></el-cascader>
				</el-form-item>
				<el-form-item label="LOGO">
					<el-upload
							action="http://127.0.0.1:9999/services/common/file"
							list-type="picture-card"
							:on-preview="handlePictureCardPreview"
							:on-remove="handleRemove"
							:on-success="handleSuccess"
							:before-upload="handleBeforeUpload"
							:file-list="fileList"
					>
						<i class="el-icon-plus"></i>
					</el-upload>
					<el-dialog v-model="dialogVisible" size="small">
						<img width="100%" :src="dialogImageUrl" alt="">
					</el-dialog>
				</el-form-item>
				<el-form-item label="描述">
					<el-input type="textarea" v-model="addForm.description"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="addFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';

	export default {
		data() {
			return {
				filters: {
					keyword: ''
				},
				brands: [],
				total: 0,
				page: 1,
				size:10,
				listLoading: false,
				sels: [],//列表选中列

				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '请输入商品名称', trigger: 'blur' }
					],
					englishName: [
						{ required: true, message: '请输入英文名称', trigger: 'blur' }
					],
					selectedOptions: [
						{ type:'array', required: true, message: '请选择商品类型', trigger: 'blur' }
					]
				},
				//编辑界面数据
				editForm: {
					id: 0,
					name: '',
					englishName: '',
					productType: '',
					description: '',
					selectedOptions: []
				},
				options:[],
				defaultParams: {
					label: 'name',
					value: 'id',
					children: 'children'
				},
				addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						{ required: true, message: '请输入商品名称', trigger: 'blur' }
					],
					englishName: [
						{ required: true, message: '请输入英文名称', trigger: 'blur' }
					],
					selectedOptions: [
						{ type:'array', required: true, message: '请选择商品类型', trigger: 'blur' }
					]
				},
				//新增界面数据
				addForm: {
					name: '',
					englishName: '',
					description: '',
					selectedOptions: []
				},
				dialogImageUrl: '',
				dialogVisible: false,
				fileList:[]
			}
		},
		methods: {
			handleCurrentChange(val) {
				this.page = val;
				this.getbrands();
			},
			//获取用户列表
			getbrands() {
				let para = {
					pageNum: this.page,
					pageSize: this.size,
					keyword: this.filters.keyword
				};
				this.listLoading = true;
				this.$http.post("/product/brand/json",para).then(res=>{
					this.listLoading = false;
					let {total,rows} = res.data;
					this.brands = rows;
					this.total = total;
				})
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					this.removeFile(row.logo);
					this.$http.delete("/product/brand/delete/"+row.id).then(res=>{
						this.listLoading = false;
						let {success,message} = res.data;
						if (success){
							this.$message({
								message: '删除成功',
								type: 'success'
							});
							this.getbrands();
						}else {
							this.$message({
								message: message,
								type: 'error'
							});
							this.getbrands();
						}
					})
				}).catch(() => {

				});
			},
			//显示编辑界面
			handleEdit: function (index, row) {
				this.editFormVisible = true;
				this.editForm = Object.assign({}, row);
				this.fileList=[];
				this.fileList.push({"url":"http://172.16.4.153/"+row.logo})
				this.editForm.selectedOptions = this.getTreeDeepArr(row.productTypeId, this.options);
			},
			//显示新增界面
			handleAdd: function () {
				this.addFormVisible = true;
				this.fileList=[];
				this.addForm = {
					name: '',
					englishName: '',
					description: '',
					selectedOptions: []
				};
			},
			//编辑
			editSubmit: function () {
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							let para = Object.assign({}, this.editForm);
							para.productTypeId = para.selectedOptions[para.selectedOptions.length-1];
							this.$http.post("/product/brand/add",para).then(res=>{
								this.editLoading = false;
								let {success,message} = res.data;
								if(success){
									this.$message({
										message: '提交成功',
										type: 'success'
									});
								}else {
									this.$message({
										message: message,
										type: 'error'
									});
								}
								this.editFormVisible = false;
								this.getbrands();
							})
						});
					}
				});
			},
			//新增
			addSubmit: function () {
				this.$refs.addForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.addLoading = true;
							let para = Object.assign({}, this.addForm);
							para.productTypeId = para.selectedOptions[para.selectedOptions.length-1];
							this.$http.post("/product/brand/add",para).then(res=>{
								this.addLoading = false;
								let {success,message} = res.data;
								if(success){
									this.$message({
										message: '提交成功',
										type: 'success'
									});
								}else {
									this.$message({
										message: message,
										type: 'error'
									});
								}
								this.addFormVisible = false;
								this.getbrands();
							})
						});
					}
				});
			},
			selsChange: function (sels) {
				this.sels = sels;
			},
			//批量删除
			batchRemove: function () {
				console.debug(this.sels);
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					this.$http.delete("/product/brand/batchdelete/"+ids).then(res=>{
						this.listLoading = false;
						let {success,message} = res.data;
						if (success){
							this.$message({
								message: '删除成功',
								type: 'success'
							});
							this.getbrands();
						}else {
							this.$message({
								message: message,
								type: 'error'
							});
							this.getbrands();
						}
					})
				}).catch(() => {

				});
			},
			getProductType(){
				this.$http.post("/product/productType/loadTree")
						.then(res=>{
							this.options=this.getTreeData(res.data);
						});
			},
			getTreeData(data){
				// 循环遍历json数据
				for(var i=0;i<data.length;i++){
					if(data[i].children.length<1){
						// children若为空数组，则将children设为undefined
						data[i].children=undefined;
					}else {
						// children若不为空数组，则继续 递归调用 本方法
						this.getTreeData(data[i].children);
					}
				}
				return data;
			},
			getTreeDeepArr(key, treeData) {
				let arr = []; // 在递归时操作的数组
				let returnArr = []; // 存放结果的数组
				let depth = 0; // 定义全局层级
				// 定义递归函数
				function childrenEach(childrenData, depthN) {
					for (var j = 0; j < childrenData.length; j++) {
						depth = depthN; // 将执行的层级赋值 到 全局层级
						arr[depthN] = (childrenData[j].id);
						if (childrenData[j].id == key) {
							returnArr = arr.slice(0, depthN+1); //将目前匹配的数组，截断并保存到结果数组，
							break
						} else {
							if (childrenData[j].children) {
								depth ++;
								childrenEach(childrenData[j].children, depth);
							}
						}
					}
					return returnArr;
				}
				return childrenEach(treeData, depth);
			},
			handleRemove(file, fileList) {
				console.debug(file);
				let filepath = '';
				if (file.size){
					filepath = file.response.result;
				}else {
					filepath = file.url.substring(20,file.url.length);
				}
				this.removeFile(filepath);
				this.fileList=[];
			},
			removeFile(path){
				this.$http.delete("/common/file?filePath="+path).then(res=>{
					let {success,message} = res.data
					if (success){
						this.$message({
							message: '删除成功',
							type: 'success'
						});
					}
				})
			},
			handlePictureCardPreview(file) {
				this.dialogImageUrl = file.url;
				this.dialogVisible = true;
			},
			handleSuccess(response, file, fileList){
				let{success,message,result}=response;
				if (success){
					this.addForm.logo=result;
					this.editForm.logo=result;
					this.$message({
						message: '上传成功',
						type: 'success'
					});
				}else {
					this.$message({
						message: message,
						type: 'error'
					});
				}
				this.fileList=fileList;
			},
			handleBeforeUpload(file){
				//看fileList中的元素个数
				if(this.fileList.length>0){
					this.$message({
						message: '只能上传一张logo图片', type: 'warning'
					});
					return false;
				}
				const isJPG = file.type.split('/')[0] == 'image';
				const isLt2M = file.size / 1024 / 1024 < 2;
				if (!isJPG) {
					this.$message.error('上传文件只能是图片!');
				}
				if (!isLt2M) {
					this.$message.error('上传头像图片大小不能超过 2MB!');
				}
				return isJPG && isLt2M;
			}
		},
		mounted() {
			this.getbrands();
			this.getProductType();
		}
	}

</script>

<style scoped>

</style>