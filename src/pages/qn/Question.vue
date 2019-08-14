<!-- 
	问题：
		1. 表单验证中如何验证题目选项
		2. 若题目名称内容过多，如何处理？
		3. 题目名称前如何添加序号？
			$index+1
 -->

<template>
	<div class="question">
		<!-- 题库管理 -->
		<!-- 按钮区 -->
		<div class="question-btns">
			<el-button size='mini' @click='toAddQuestion'>添加</el-button>
			<el-button size='mini' @click='batchDeleteQuestion'>批量删除</el-button>
		</div>
		<!-- 内容区 -->
		<!-- {{ids}} -->
		<div class="question-content" v-loading='loading'>
			<ul>
				<li class="question-list" v-for='(q,$index) in questions'>
					<div class="question-title">
						<el-checkbox v-model="ids" :label='q.id'>{{$index+1}}. {{q.name}}</el-checkbox>
						<el-tag type="warning" size='mini'>{{q.questionType}}</el-tag>
					</div>
					<div class="question-options" v-if='q.questionType!="简答题"'>
						<ol>
							<li v-for='o in q.options'>
								<span>{{o.label}}:</span>
								<span> {{o.name}}</span>
							</li>
						</ol>
					</div>
					<div class="question-set">
						<i class="fa fa-pencil" title="修改" @click='toUpdateQuestion(q)'></i>
						<i class="fa fa-trash" title="删除" @click='deleteQuestion(q.id)'></i>
					</div>
				</li>
			</ul>
		</div>
		<!-- 模态框 -->
		<el-dialog
		  :title='questionDialog.title'
		  :visible.sync="questionDialog.visible"
		  width="50%"
		  :before-close="closeModal">
		  <!-- {{questionDialog.form}} -->
		  <el-form :model="questionDialog.form" size='mini' label-position='left' label-width="6em" :rules="rules" ref="ruleForm">
		    <el-form-item label="题目名称" prop='name'>
		      <el-input v-model="questionDialog.form.name" type='textarea' :rows='2' placeholder='请输入题目名称'></el-input>
		    </el-form-item>
				<el-form-item label="题目类型" prop='questionType'>
			    <el-select v-model="questionDialog.form.questionType" placeholder="请选择题目类型" style='width:50%'>
			      <el-option label="单选题" value="单选题"></el-option>
			      <el-option label="多选题" value="多选题"></el-option>
			      <el-option label="简答题" value="简答题"></el-option>
			    </el-select>
			  </el-form-item>
		    <el-form-item label="题目选项" v-if='questionDialog.form.questionType != "简答题"' prop="option">
		      <el-table :data='questionDialog.form.options' size='mini' border width='100%'>
		      	<el-table-column label='序号' width='60' align='center'>
		      		<template slot-scope='scope'>
		      			{{scope.$index+1}}
		      		</template>
		      	</el-table-column>
		      	<el-table-column label='label' prop='label' width='60' align='center'>
		      		
		      	</el-table-column>
		      	<el-table-column label='选项' align='center'>
		      		<template slot-scope='scope'>
		      			<el-input size='mini' v-model='scope.row.name'></el-input>
		      		</template>
		      	</el-table-column>
		      	<el-table-column label='分值' prop='' width='60' align='center'>
		      		<template slot-scope='scope'>
		      			<el-input size='mini' v-model='scope.row.score'></el-input>
		      		</template>
		      	</el-table-column>
		      	<el-table-column label='操作' width='60' align='center'>
		      		<template slot-scope='scope'>
		      			<i class="fa fa-trash" @click='deleteOption(scope.row)'></i>
		      		</template>
		      	</el-table-column>
		      </el-table>
		      <div style="text-align:right"  title="添加">
		      	<el-button size='mini' @click='addOption' :disabled='questionDialog.form.options.length == 5'>
		      		<i class='fa fa-plus'></i>
		      	</el-button>
		      </div>
		    </el-form-item>
		  </el-form>
		  <span slot="footer" class="dialog-footer">
		    <el-button size='mini' @click="closeModal">取 消</el-button>
		    <el-button type="primary" size='mini' @click="saveOrUpdateQuestion">确 定</el-button>
		  </span>
		</el-dialog>
	</div>
</template>
<script>
	import getAxios from '@/http/getAxios'
	let axios = getAxios()
	let axios_array = getAxios('array')
	export default {
		data(){
			return {
				questions:[],
				loading:false,
				ids:[],
				questionDialog:{
					title:'',
					visible:false,
					form:{
						options:[]
					}
				},
				rules:{
					name:[
						{ required: true, message: '请输入题目名称', trigger: 'blur' },
						{ min: 1, max: 130, message: '长度在 1 到 130 个字符', trigger: 'blur' }
					],
					questionType:[
						{ required: true, message: '请选择题目类型', trigger: 'change' }
					]
				}
			}
		},
		methods:{
			// 批量删除题目
			batchDeleteQuestion(){
				this.$confirm('此操作将永久删除该题目, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        .then(()=>{
        	axios.post('/question/batchDeleteQuestion',{ids:this.ids})
        	.then((data)=>{
        		console.log(data)
        		this.$message({
            	type: 'success',
            	message: '删除成功!'
          	});
          	this.findAllQuestion();
        	})
        	.catch(()=>{
        		this.$message.error('网络异常');
        	})
        })
			},
			// 删除题目选项
			deleteOption(row){
				_.remove(this.questionDialog.form.options,(item)=>{
					return item == row
				})
				this.questionDialog.form.options.push(1);
				this.questionDialog.form.options.pop();
			},
			// 删除题目
			deleteQuestion(id){
				this.$confirm('此操作将永久删除该题目, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        .then(()=>{
        	axios.get('/question/deleteQuestionById?id='+id)
        	.then(()=>{
        		this.$message({
            	type: 'success',
            	message: '删除成功!'
          	});
          	this.findAllQuestion();
        	})
        	.catch(()=>{
        		this.$message.error('网络异常');
        	})
        })
			},
			// 修改题目信息
			toUpdateQuestion(q){
				this.questionDialog.title = '修改题目信息';
				this.questionDialog.form = q;
				this.questionDialog.visible = true;
			},
			// 新增题目信息
			toAddQuestion(){
				this.questionDialog.form = {
					options:[]
				};
				this.questionDialog.title = '添加题目信息';
				this.questionDialog.visible = true;
			},
			// 保存题目
			saveOrUpdateQuestion(){
				this.$refs.ruleForm.validate((valid)=>{
					if(valid){
						axios_array.post('/question/saveOrUpdateQuestion',this.questionDialog.form)
						.then(({data:result})=>{
							// console.log(result)
							if(result.status == 200){
								this.$message({
		          	message: '保存成功',
		          	type: 'success'
		        	});
		        	this.findAllQuestion();
		        	this.closeModal()
							}
						})
						.catch(()=>{
							this.$message.error('网络异常');
						})
					}
				})
			},
			// 加载数据
			findAllQuestion(){
				this.loading = true;
				axios.get('/question/findAllQuestionVM')
				.then(({data:result})=>{
					this.questions = result.data
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
				.finally(()=>{
					this.loading = false;
				})
			},
			// 关闭模态框
			closeModal(){
				this.questionDialog.visible = false;
			},
			// 添加题目选项
			addOption(){
				let option = {};
				let label = '';
				let score = 0;
				switch(this.questionDialog.form.options.length){
					case 0:
						label = 'A';
						score = 5;
						break;
					case 1:
						label = 'B';
						score = 4;
						break;
					case 2:
						label = 'C';
						score = 3;
						break;
					case 3:
						label = 'D';
						score = 2;
						break;
					case 4:
						label = 'E';
						score = 1;
						break;
				}
				option.label = label;
				option.score = score;
				this.questionDialog.form.options.push(option)
			}
		},
		created(){
			this.findAllQuestion()
		}
	}
</script>
<style>
	.question {
		padding: 1em;
	}
	.question-btns {
		text-align: right;
		margin-bottom: 1em;
	}
	.question-list {
		border-bottom: 1px solid #ededed;
		padding: .5em 0;
		position: relative;
	}
	.question-list .question-title {
		line-height: 2em;
		color: #333;
	}
	.question-list .question-options {
		padding-left: 2em;
	}
	.question-list .question-options li {
		line-height: 2em;
	}
	.question-list .question-set {
		position: absolute;
		right: .5em;
		bottom: .5em;
	}
	i.fa {
		cursor: pointer;
		margin: 0 .3em; 
	}
	i.fa-pencil {
		color: #409EFF;
	}
	i.fa-trash {
		color: #F56C6C
	}
</style>