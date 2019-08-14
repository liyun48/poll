<template>
	<div class="clazz">
		<!-- 班级管理 -->
		<!-- 按钮区 -->
		<div class="clazz_btns">
			<el-button size='mini' @click='toAddClazz'>添加</el-button>
			<el-button size='mini' @click='batchDeleteClazz'>批量删除</el-button>
		</div>
		<!-- 数据内容 -->
		<div class="user_table" v-loading="loading">
			<el-table
		    :data="clazz"
		    border
		    style="width: 100%"
		    @selection-change="handleSelectionChange"
		    size='mini'>
		    <el-table-column
		      type="selection"
		      width="50"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="name"
		      label="班级名称"
		      width="180"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="description"
		      label="班级简介"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="charge.name"
		      label="班主任"
		      width="180"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		    	label="操作"
		    	width='80'
		    	align='center'>
					<template slot-scope="{row}">
						<i class="fa fa-pencil" size='mini' @click='toUpdateClazz(row)'></i>
						<i class="fa fa-trash" size='mini' @click='deleteClazz(row.id)'></i>
      		</template>
		    </el-table-column>
		  </el-table>
			<!-- {{multipleSelection}} -->
		</div>
		<!-- 模态框 -->
		<el-dialog :title="clazzDialog.title" :visible.sync="clazzDialog.visible" :before-close='closeModal'>
		  <el-form :model="clazzDialog.form" label-width='6em' size='mini'>
		    <el-form-item label="所属年级">
		    <el-select v-model="clazzDialog.form.gradeId" placeholder="请选择所属年级">
		        <el-option v-for='g in grades' :key="g.id" :label="g.name" :value='g.id'></el-option>
		      </el-select>
		    </el-form-item>
		    <el-form-item label="班级名称">
		      <el-input v-model="clazzDialog.form.name" autocomplete="off"></el-input>
		    </el-form-item>
		    <el-form-item label="班主任">
		      <el-select v-model="clazzDialog.form.chargeId" placeholder="请选择班主任">
		        <el-option v-for='u in users' :key="u.id" :label="u.name" :value='u.id'></el-option>
		      </el-select>
		    </el-form-item>
		    <el-form-item label="班级介绍">
		      <el-input v-model="clazzDialog.form.description" type='textarea' :rows='2' placeholder='请输入班级介绍'></el-input>
		    </el-form-item>
		  </el-form>
		  <!-- {{clazzDialog.form}} -->
		  <div slot="footer" class="dialog-footer">
		    <el-button @click="closeModal" size='mini'>取 消</el-button>
		    <el-button type="primary" @click="saveOrUpdateClazz" size='mini'>确 定</el-button>
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
				clazz:[],
				grades:[],
				users:[],
				multipleSelection:[],
				loading:false,
				clazzDialog:{
					title:'',
					visible:false,
					form:{}
				}
			}
		},
		methods:{
			// 保存班级信息
			saveOrUpdateClazz(){
						axios.post('/clazz/saveOrUpdateClazz',this.clazzDialog.form)
						.then(()=>{
							this.$message({
		          	message: '保存成功',
		          	type: 'success'
		        	});
		        	this.findAllClazz();
		        	this.closeModal();
						})
						.catch(()=>{
							this.$message({
		          	message: '保存失败',
		          	type: 'error'
		        	});
						})
			},
			// 关闭模态框
			closeModal(){
				this.clazzDialog.visible = false;
			},
			// 删除班级信息
			deleteClazz(id){
				this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
		          confirmButtonText: '确定',
		          cancelButtonText: '取消',
		          type: 'warning'
		        })
		        .then(()=>{
		        	axios.get('/clazz/deleteClazzById',{
								params:{
									id:id
								}
							})
							.then(()=>{
								this.$message({
		          		message: '删除成功',
		          		type: 'success'
		        		});
		        		this.findAllClazz()
							})
							.catch(()=>{
								this.$message({
		          		message: '删除失败',
		          		type: 'error'
		        		});
							})
		        })
			},
			// 批量删除班级信息
			batchDeleteClazz(){
				this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        .then(()=>{
        	let ids = this.multipleSelection.map((item)=>{
        		return item.id
        	});
        	// console.log(ids)
        	axios.post('/clazz/batchDeleteClazz',{ids})
					.then(()=>{
						this.$message({
          		message: '删除成功',
          		type: 'success'
        		});
        		this.findAllClazz()
					})
					.catch((error)=>{
						this.$message({
          		message: '删除失败',
          		type: 'error'
        		});
        		console.log(error)
					})
        })
			},
			// 修改班级信息
			toUpdateClazz(row){
				this.clazzDialog.title = '修改班级';
				let cla = _.clone(row);
				cla.gradeId = cla.grade.id;
				delete cla.grade;
				cla.chargeId = cla.charge.id;
				delete cla.charge;
				// console.log(cla)
				this.clazzDialog.form = cla;
				this.clazzDialog.visible = true;
			},
			// 添加班级信息
			toAddClazz(){
				this.clazzDialog.title = '添加班级';
				this.clazzDialog.form = {};
				this.clazzDialog.visible = true;
			},
			// 加载班级信息
			findAllClazz(){
				this.loading = true;
				axios.get('/clazz/findAllVM')
				.then(({data:result})=>{
					this.clazz = result.data
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
				.finally(()=>{
					this.loading = false;
				})
			},
			// 加载班级信息
			findAllGrade(){
				axios.get('/grade/findAll')
				.then(({data:result})=>{
					this.grades = result.data
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
			},
			// 加载讲师信息
			findAllUser(){
				axios.get('/user/findAll')
				.then(({data:result})=>{
					this.users = result.data
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
			},
			// 多选框
			handleSelectionChange(val){
				this.multipleSelection = val;
			}
		},
		created(){
			this.findAllClazz();
			this.findAllGrade();
			this.findAllUser();
		}
	}
</script>
<style>
	.clazz {
		padding: 1em;
	}
	.clazz_btns {
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