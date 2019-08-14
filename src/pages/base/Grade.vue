<template>
	<div class="grade">
		<!-- 年级管理 -->
		<!-- 按钮 -->
		<div class="grade_btns">
			<el-button size='mini' @click='toAddGrade'>添加</el-button>
			<el-button size='mini' @click='batchDeleteGrades'>批量删除</el-button>
		</div>
		<!-- 表格 -->
		<div v-loading='loading'>
			<el-table :border='true' size='mini' :data="grades" style="width: 100%" @selection-change="handleSelectionChange">
				<el-table-column type="selection" width="50" fixed  align='center'></el-table-column>
	      <el-table-column prop="name" label="名称" width="160" align='center'></el-table-column>
	      <el-table-column prop="description" label="简介" align='center'></el-table-column>
	      <el-table-column label="操作" width='70' fixed="right" align='center'>
	      	<template slot-scope='{row}'>
	      		<i class="fa fa-trash" @click='deleteGrade(row.id)'></i>
	      		<i class="fa fa-pencil" @click='toUpdateGrade(row)'></i>
	      	</template>
	      </el-table-column>
	    </el-table>
		</div>
		<!-- 模态框 -->
		<el-dialog :title="gradeDialog.title" :visible.sync="gradeDialog.visible">
			<!-- {{gradeDialog.form}} -->
		  <el-form ref="gradeForm" :rules='rules' :model="gradeDialog.form" label-position='left' size='small'>
		    <el-form-item label="年级名称" label-width="100px" prop="name">
		      <el-input v-model="gradeDialog.form.name" autocomplete="off"></el-input>
		    </el-form-item>
			<el-form-item label="年级简介" label-width="100px" prop="descriptioin">
		      <el-input
					  type="textarea"
					  :rows="2"
					  placeholder="请输入内容"
					  v-model="gradeDialog.form.descriptioin">
					</el-input>
		    </el-form-item>
		  </el-form>
		  <div slot="footer" class="dialog-footer">
		    <el-button size='mini' @click='closeGradeDialog'>取 消</el-button>
		    <el-button size='mini' type="primary" @click='saveOrUpdateGrade'>确 定</el-button>
		  </div>
		</el-dialog>
	</div>
</template>
<script>
	import getAxios from '@/http/getAxios'
	let axios = getAxios();
	export default {
		data(){
			return {
				loading:false,
				grades:[],
				rules:{
					name:[{
						required: true, 
						message: '请输入栏目名称',
						trigger: 'blur' 
					}],
					descriptioin:[{
						required: true, 
						message: '请输入介绍信息',
						trigger: 'blur' 
					}]
				},
				multipleSelection:[],
				gradeDialog:{
					title:'',
					visible:false,
					form:{}
				}
			}
		},
		created(){
			this.findAllGrades();
		},
		methods:{
			// 加载数据
			findAllGrades(){
				this.loading = true;
				axios.get('/grade/findAll')
				.then(({data:result})=>{
					this.grades = result.data;
				})
				.finally(()=>{
					this.loading = false;
				});
			},
			// 添加
			toAddGrade(){
				this.gradeDialog.title = '添加年级';
				this.gradeDialog.visible = true;
			},
			// 关闭模态框
			closeGradeDialog(){
				this.gradeDialog.visible = false;
				this.gradeDialog.form = {};
				this.$refs.gradeForm.resetFields();
			},
			// 保存
			saveOrUpdateGrade(){
				this.$refs.gradeForm.validate((valid)=>{
					if(valid){
						axios.post('/grade/saveOrUpdate',this.gradeDialog.form)
						.then(({data:result})=>{
							this.findAllGrades();
							this.closeGradeDialog();
							this.$message({
			          		message: '保存成功',
			          		type: 'success'
			        		});
						})
						.catch(()=>{
							this.$message({
				          		message: '服务器异常',
				          		type: 'error'
				        	});
						})		
					}
				});
			},
			// 修改
			toUpdateGrade(row){
				this.gradeDialog.title = '修改年级';
				this.gradeDialog.visible = true;
				this.gradeDialog.form = row;
			},
			// 批量删除
			batchDeleteGrades(){
				this.$confirm('此操作将永久删除该数据?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(()=>{
        	let ids = this.multipleSelection.map((item)=>{
						return item.id;
					})
					axios.post('/grade/batchDelete',{ids})
					.then(({data:result})=>{
						this.findAllGrades();
						this.$message({
			          		message: '删除成功',
			          		type: 'success'
			        		});
					})
					.catch(()=>{
						this.$message({
		          		message: '服务器异常',
		          		type: 'error'
		        		});
					})
        })
			},
			handleSelectionChange(val){
				this.multipleSelection = val;
			},
			// 删除
			deleteGrade(id){
				this.$confirm('此操作将永久删除该数据?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(()=>{
        	axios.get('/grade/deleteById?id='+id)
					.then(({data:result})=>{
						this.findAllGrades();
						this.$message({
			          		message: '删除成功',
			          		type: 'success'
			        		});
					})
					.catch(()=>{
						this.$message({
		          		message: '服务器异常',
		          		type: 'error'
		        		});
					})
        })
			}
		}
	}
</script>
<style>
	.grade {
		padding: 1em;
	}
	.grade_btns {
		margin-bottom: 1em;
		text-align: right;
	}
	i.fa {
		margin: 0 .5em;
		cursor: pointer;
	}
	i.fa-pencil {
		color: #409EFF;
	}
	i.fa-trash {
		color: #F56C6C;
	}
</style>