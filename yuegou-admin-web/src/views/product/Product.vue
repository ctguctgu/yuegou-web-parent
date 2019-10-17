<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="关键字"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getProducts">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleViewProperties">显示属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleSkuProperties">SKU属性</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleOnSale">上架</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleOffSale">下架</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="products" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="55">
			</el-table-column>
			<el-table-column prop="name" label="标题" width="150" sortable>
			</el-table-column>
			<el-table-column prop="subName" label="副标题" width="130" sortable>
			</el-table-column>
			<el-table-column prop="productType.name" label="商品类型" width="120" sortable>
			</el-table-column>
			<el-table-column prop="brand.name" label="品牌" width="100" sortable>
			</el-table-column>
			<el-table-column prop="onSaleTime" label="上架时间" width="165" :formatter="formatOnSale" sortable>
			</el-table-column>
			<el-table-column prop="offSaleTime" label="下架时间" width="165" :formatter="formatOffSale" sortable>
			</el-table-column>
			<el-table-column prop="state" label="状态" min-width="80" sortable>
				<template scope="scope">
					<span v-if="scope.row.state == 1" style="color: greenyellow">上架</span>
					<span v-else style="color: orangered">下架</span>
				</template>
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
				<el-form-item label="标题" prop="name">
					<el-input v-model="editForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="副标题" prop="subName">
					<el-input v-model="editForm.subName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品类型">
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
				<el-form-item label="品牌">
					<el-select v-model="editForm.brandId" clearable placeholder="请选择品牌">
						<el-option
								v-for="item in brands"
								:label="item.name"
								:value="item.id">
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="媒体属性">
					<el-upload
							action="http://127.0.0.1:9999/services/common/file"
							list-type="picture-card"
							:on-preview="handlePictureCardPreview"
							:on-remove="handleRemove"
							:on-success="handleSuccess"
							:file-list="fileList"
					>
						<i class="el-icon-plus"></i>
					</el-upload>
					<el-dialog v-model="dialogVisible" size="small">
						<img width="100%" :src="dialogImageUrl" alt="">
					</el-dialog>
				</el-form-item>
				<el-form-item label="商品描述">
					<el-input type="textarea" v-model="editForm.ext.description" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品详情">
					<quill-editor
							ref="QuillEditor"
							v-model="editForm.ext.richContent"
							:options="editorOption"
					></quill-editor>
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
				<el-form-item label="标题" prop="name">
					<el-input v-model="addForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="副标题" prop="subName">
					<el-input v-model="addForm.subName" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品类型">
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
				<el-form-item label="品牌">
					<el-select v-model="addForm.brandId" clearable placeholder="请选择品牌">
						<el-option
								v-for="item in brands"
								:label="item.name"
								:value="item.id">
						</el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="媒体属性">
					<el-upload
							action="http://127.0.0.1:9999/services/common/file"
							list-type="picture-card"
							:on-preview="handlePictureCardPreview"
							:on-remove="handleRemove"
							:on-success="handleSuccess"
							:file-list="fileList"
					>
						<i class="el-icon-plus"></i>
					</el-upload>
					<el-dialog v-model="dialogVisible" size="small">
						<img width="100%" :src="dialogImageUrl" alt="">
					</el-dialog>
				</el-form-item>
				<el-form-item label="商品描述">
					<el-input type="textarea" v-model="addForm.ext.description" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="商品详情">
					<quill-editor
							ref="QuillEditor"
							v-model="addForm.ext.richContent"
							:options="editorOption"
					></quill-editor>
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
				editorOption:{
					modules:{
						toolbar:[
							['bold', 'italic', 'underline', 'strike'],
							['blockquote', 'code-block'],
							[{ 'header': 1 }, { 'header': 2 }],
							[{ 'list': 'ordered'}, { 'list': 'bullet' }],
							[{ 'script': 'sub'}, { 'script': 'super' }],
							[{ 'indent': '-1'}, { 'indent': '+1' }],
							[{ 'direction': 'rtl' }],
							[{ 'size': ['small', false, 'large', 'huge'] }],
							[{ 'header': [1, 2, 3, 4, 5, 6, false] }],
							[{ 'color': [] }, { 'background': [] }],
							[{ 'font': ['SimSun', 'SimHei','Microsoft-YaHei','KaiTi','FangSong','Arial','Times-New-Roman','sans-serif'] }],
							[{ 'align': [] }],
							['clean'],
							['image','video']
						]
					}
				},
				options:[],
				dialogImageUrl: '',
				dialogVisible: false,
				defaultParams: {
					label: 'name',
					value: 'id',
					children: 'children'
				},
				fileList:[],
				brands:[],
				filters: {
					keyword: ''
				},
				products: [],
				total: 0,
				page: 1,
				size:10,
				listLoading: false,
				sels: [],//列表选中列
				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '请输入标题', trigger: 'blur' }
					],
					subName: [
						{ required: true, message: '请输入副标题', trigger: 'blur' }
					]
				},
				//编辑界面数据
				editForm: {
					name: '',
					subName: '',
					productTypeId: null,
					brandId: null,
					medias:'',
					selectedOptions: [],
					ext:{
						description:'',
						richContent:''
					}
				},
				addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						{ required: true, message: '请输入标题', trigger: 'blur' }
					],
					subName: [
						{ required: true, message: '请输入副标题', trigger: 'blur' }
					]
				},
				//新增界面数据
				addForm: {
					name: '',
					subName: '',
					productTypeId: null,
					brandId: null,
					medias:'',
					selectedOptions: [],
					ext:{
						description:'',
						richContent:''
					}
				}
			}
		},
		methods: {
			//显示属性维护
			handleViewProperties(){},
			//sku属性维护
			handleSkuProperties(){},
			//上架
			handleOnSale(){},
			//下架
			handleOffSale(){},
			formatOnSale(row,colunm){
				return this.formatDate(row.onSaleTime);
			},
			formatOffSale(row,colunm){
				return this.formatDate(row.offSaleTime);
			},
			//时间显示转换
			formatDate(time) {
				if (!time){
					return null;
				}
				let getDate = new Date(time);
				let year = getDate.getFullYear();
				let month = getDate.getMonth()+1;
				let day = getDate.getDate();
				let hours = getDate.getHours();
				let minutes = getDate.getMinutes();
				let seconds = getDate.getSeconds();
				let result = year+"-"+this.formatTimeNum(month)+"-"+this.formatTimeNum(day)+" "
						+this.formatTimeNum(hours)+":"+this.formatTimeNum(minutes)+":"+this.formatTimeNum(seconds);
				return result;
			},
			formatTimeNum(num){
				if(num>=10){
					return num;
				}
				return "0"+num;
			},
			handleCurrentChange(val) {
				this.page = val;
				this.getProducts();
			},
			//获取用户列表
			getProducts() {
				let para = {
					pageNum: this.page,
					pageSize: this.size,
					keyword: this.filters.keyword
				};
				this.listLoading = true;
				this.$http.post("/product/product/json",para).then(res=>{
					this.listLoading = false;
					let {total,rows} = res.data;
					this.products = rows;
					this.total = total;
				})
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					if (row.medias){
						let s = row.medias.split(",");
						for (var i =0;i<s.length;i++){
							this.removeFile(s[i]);
						}
					}
					this.$http.delete("/product/product/delete/"+row.id).then(res=>{
						this.listLoading = false;
						let {success,message} = res.data;
						if (success){
							this.$message({
								message: '删除成功',
								type: 'success'
							});
							this.getProducts();
						}else {
							this.$message({
								message: message,
								type: 'error'
							});
							this.getProducts();
						}
					})
				}).catch(() => {

				});
			},
			//显示编辑界面
			handleEdit: function (index, row) {
				this.editFormVisible = true;
				this.editForm = Object.assign({}, row);
				this.$http.get("/product/productExt/findByProduct?id="+row.id).then(res=>{
					this.editForm.ext=res.data;
				})
				this.fileList=[];
				let ary = row.medias.split(",");
				for (var i =0;i<ary.length;i++){
					this.fileList.push({"url":"http://172.16.4.153/"+ary[i]})
				}
				this.editForm.selectedOptions = this.getTreeDeepArr(row.productTypeId, this.options);
			},
			//显示新增界面
			handleAdd: function () {
				this.addFormVisible = true;
				this.addForm = {
					name: '',
					subName: '',
					productTypeId: null,
					brandId: null,
					medias:'',
					selectedOptions: [],
					ext:{
						description:'',
						richContent:''
					}
				};
			},
			//编辑
			editSubmit: function () {
				var length = this.fileList.length;
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							let para = Object.assign({}, this.editForm);
							para.productTypeId = para.selectedOptions[para.selectedOptions.length-1];
							console.debug(this.fileList);
							var path = '';
							for (var i = 0; i<length;i++){
								if (this.fileList[i].size){
									path += this.fileList[i].response.result +',';
								}else {
									path += this.fileList[i].url.substring(20,this.fileList[i].url.length) +',';
								}
							}
							para.medias = path.slice(0,path.length-1);
							this.$http.post("/product/product/add",para).then(res=>{
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
								this.$refs['editForm'].resetFields();
								this.editFormVisible = false;
								this.getProducts();
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
							para.medias = this.fileList.map(file=>file.response.result).join(",");
							this.$http.post("/product/product/add",para).then(res=>{
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
								this.$refs['addForm'].resetFields();
								this.addFormVisible = false;
								this.fileList = [];
								this.getProducts();
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
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					this.$http.delete("/product/product/batchdelete/"+ids).then(res=>{
						this.listLoading = false;
						let {success,message} = res.data;
						if (success){
							this.$message({
								message: '删除成功',
								type: 'success'
							});
							this.getProducts();
						}else {
							this.$message({
								message: message,
								type: 'error'
							});
							this.getProducts();
						}
					})
				}).catch(() => {

				});
			},
			getBrands(){
				this.$http.get("/product/brand/list").then(res=>{
					this.brands = res.data;
				})
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
				let filepath = '';
				if (file.size){
					filepath = file.response.result;
				}else {
					filepath = file.url.substring(20,file.url.length);
				}
				this.removeFile(filepath);
				this.fileList = fileList;
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
			}
		},
		mounted() {
			this.getProducts();
			this.getProductType();
			this.getBrands();
		}
	}

</script>

<style scoped>

</style>