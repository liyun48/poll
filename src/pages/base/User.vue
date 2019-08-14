<template>
	<div class="user">
	<!-- 用户管理 -->
		<div class="user_info">
			<el-input
  			placeholder="请输入关键字"
  			prefix-icon="el-icon-search"
  			size='mini'
  			style="width:150px"
  			v-model="keywords"
  			clearable>
			</el-input>
			<!-- {{keywords}} -->
		</div>
		<!-- 按钮区 -->
		<div class="user_btns">
			<el-button size='mini' @click='toAddUser'>添加</el-button>
			<el-button size='mini' @click='batchDeleteUser'>批量删除</el-button>
		</div>
		<!-- 数据内容 -->
		<div class="user_table" v-loading="loading">
		<!-- {{multipleSelection}} -->
			<el-table
		    :data="users"
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
		      label="教师姓名"
		      width="180"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="gender"
		      label="性别"
		      width="180"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="type"
		      label="类型"
		      width="180"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="birth"
		      label="出生日期"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="hiredate"
		      label="入职时间"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		    	label="操作"
		    	width='70'
		    	align='center'>
					<template slot-scope="{row}">
						<i class="fa fa-pencil" size='mini' @click='toUpdateUser(row)'></i>
						<i class="fa fa-trash" size='mini' @click='deleteUser(row.id)'></i>
      		</template>
		    </el-table-column>
		  </el-table>
		  <!-- {{users}} -->
		</div>
		<!-- 模态框 -->
		<el-dialog
		  :title='uDialog.title'
		  :visible.sync="uDialog.visible"
		  width="30%"
		  :before-close="closeModal">
		  <el-form :model="uDialog.form" size='mini' label-position='left' label-width="6em" :rules="rules" ref="ruleForm">
		    <el-form-item label="教师姓名" prop='name'>
		      <el-input v-model="uDialog.form.name" autocomplete="off"></el-input>
		    </el-form-item>
		    <el-form-item label="教师性别" prop='gender'>
		      <el-radio v-model="uDialog.form.gender" label="男">男</el-radio>
  				<el-radio v-model="uDialog.form.gender" label="女">女</el-radio>
		    </el-form-item>
				<el-form-item label="类型" prop='type'>
			    <el-select v-model="uDialog.form.type" placeholder="请选择用户类型">
			      <el-option label="班主任" value="班主任"></el-option>
			      <el-option label="讲师" value="讲师"></el-option>
			    </el-select>
			  </el-form-item>
		    <el-form-item label="出生日期" prop='birth'>
		      <el-date-picker
			      v-model="uDialog.form.birth"
			      type="date"
			      placeholder="选择日期"
			      format="yyyy-MM-dd"
      			value-format="yyyy-MM-dd">
			    </el-date-picker>
		    </el-form-item>
		    <el-form-item label="入职时间" prop='hiredate'>
		      <el-date-picker
			      v-model="uDialog.form.hiredate"
			      type="date"
			      placeholder="选择日期"
			      format="yyyy-MM-dd"
      			value-format="yyyy-MM-dd">
			    </el-date-picker>
		    </el-form-item>
		  </el-form>
  		<!-- {{uDialog.form}} -->
		  <span slot="footer" class="dialog-footer">
		    <el-button size='mini' @click="closeModal">取 消</el-button>
		    <el-button type="primary" size='mini' @click="saveOrUpdateUser">确 定</el-button>
		  </span>
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
				users:[],
				multipleSelection:[],
				uDialog:{
					visible:false,
					form:{},
					title:''
				},
				keywords:'',
				rules:{
					name:[{
						required: true,
						message: '请输入姓名',
						trigger: 'blur'
					},{
						min: 3,
						max: 9, 
						message: '长度在 3 到 9 个字符', 
						trigger: 'blur'
					}],
					birth:[{
						type: 'string',
						required: true,
						message: '请选择日期',
						trigger: 'change'
					}],
					hiredate:[{
						type: 'string',
						required: true,
						message: '请选择时间',
						trigger: 'change'
					}],
				}
			}
		},
		methods:{
			// 批量删除
			batchDeleteUser(){
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
        	axios.post('/user/batchDelete',{ids})
					.then(()=>{
						this.$message({
          		message: '删除成功',
          		type: 'success'
        		});
        		this.findAllUser()
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
			// 通过关键字查找
			findUserByKeywords(){
				axios.get('/user/query',{
					params:{
						keywords:this.keywords
					}
				})
				.then(({data:result})=>{
					// this.findAllUser()
					this.users = result.data
				})
			},
			// 删除
			deleteUser(id){
				this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        .then(()=>{
        	axios.get('/user/deleteById',{
						params:{
							id:id
						}
					})
					.then(()=>{
						this.$message({
          		message: '删除成功',
          		type: 'success'
        		});
        		this.findAllUser()
					})
					.catch(()=>{
						this.$message({
          		message: '删除失败',
          		type: 'error'
        		});
					})
        })
			},
			// 保存
			saveOrUpdateUser(){
				this.$refs.ruleForm.validate((valid)=>{
					if(valid){
						axios.post('/user/saveOrUpdate',this.uDialog.form)
						.then(()=>{
							this.$message({
		          	message: '保存成功',
		          	type: 'success'
		        	});
		        	this.findAllUser();
		        	this.closeModal();
						})
						.catch(()=>{
							this.$message({
		          	message: '保存失败',
		          	type: 'error'
		        	});
						})
					}
				})
			},
			// 添加
			toAddUser(){
				this.uDialog.title = '添加教师信息';
				this.uDialog.form = {
					gender:'男',
					type:'讲师'
				};
				this.uDialog.visible = true;
			},
			// 修改
			toUpdateUser(row){
				this.uDialog.title = '修改教师信息';
				this.uDialog.form = row;
				this.uDialog.visible = true;
			},
			// 加载数据
			findAllUser(){
				this.loading = true;
				axios.get('/user/findAll')
				.then(({data:result})=>{
					this.users = result.data
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
				.finally(()=>{
					this.loading = false;
				})
			},
			// 多选框获取数据
			handleSelectionChange(val){
				this.multipleSelection = val;
			},
			// 关闭模态框
			closeModal(){
        this.uDialog.visible = false;
			}
		},
		watch:{
			keywords(){
				this.findUserByKeywords()
			}
		},
		created(){
			this.findAllUser()
		}
	}
</script>
<style>
	.user {
		padding: 1em;
	}
	.user_info {
		float: left;
	}
	.user_btns {
		margin-bottom: 1em;
		text-align: right;
	}
	i.fa {
		cursor: pointer;
		margin: 0 .5em;
	}
	i.fa-pencil {
		color: #409EFF;
	}
	i.fa-trash {
		color: #F56C6C;
	}
</style>