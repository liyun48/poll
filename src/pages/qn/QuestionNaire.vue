<template>
	<div class="questionNaire">
		<!-- 问卷管理 -->
		<!-- 按钮区 -->
		<div class="questionNaire-btns">
			<el-button size='mini' @click='toAddQuestionNaire'>添加</el-button>
			<el-button size='mini' @click='batchDeleteQuestionNaire'>批量删除</el-button>
		</div>
		<!-- 内容区 -->
		<div class="questionNaire-table">
			<el-table
		    :data="questionNaires"
		    border
		    style="width: 100%"
		    size='mini'
		    @selection-change="handleSelectionChange"
		    v-loading='loading'>
		    <el-table-column
      		type="selection"
      		width="55"
      		align='center'>
    		</el-table-column>
		    <el-table-column
		      prop="name"
		      label="问卷名称"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="description"
		      label="问卷描述"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      label="操作"
		      width='100'
		      align='center'>
		      <template slot-scope='{row}'>
		      	<i class="fa fa-eye" title="预览" @click='toPreview(row)'></i>
		      	<i class="fa fa-pencil" title="修改" @click='toUpdateQuestionNaire(row)'></i>
						<i class="fa fa-trash" title="删除" @click='deleteQuestionNaire(row.id)'></i>
		      </template>
		    </el-table-column>
		  </el-table>
		</div>
		<!-- 模态框 -->
		<el-dialog
		  :title='questionNaireDialog.title'
		  :visible.sync="questionNaireDialog.visible"
		  fullscreen
		  :before-close="closeModal">
		  <!-- {{questionNaireDialog.form}} -->
		  <el-form :model="questionNaireDialog.form" size='mini' label-position='left' label-width="6em" :rules="rules" ref="ruleForm">
		    <el-form-item label="问卷名称" prop='name'>
		    	<el-input v-model="questionNaireDialog.form.name" placeholder='请输入问卷名称' ></el-input>
		    </el-form-item>
		    <el-form-item label="问卷描述" prop='description'>
		      <el-input v-model="questionNaireDialog.form.description" type='textarea' :rows='2' placeholder='请输入问卷描述'></el-input>
		    </el-form-item>
				<el-form-item label="题目列表">
			    <el-button @click="openInnerModal">点击选择</el-button>
			  </el-form-item>
		  </el-form>
		  <ul>
		  	<li class="question-list" v-for='(q,$index) in choosedquestions'>
					<div class="question-title">
						<span>{{$index+1}}. </span> {{q.name}}
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
						<i class="fa fa-trash" title="删除" @click='deleteQuestion(q)'></i>
					</div>
				</li>
		  </ul>
		  <el-dialog
	      width="60%"
	      title="选择题目"
	      :visible.sync="innerDialog.visible"
	      append-to-body
	      :before-close="closeInnerModal">
	      <ul class="choose-option-list">
	      	<li v-for='q in questions'>
	      		<el-checkbox v-model="choosedquestions" :label='q'>{{q.name}}</el-checkbox>
      		</li>
	      </ul>
     		{{choosedquestions}}
				<span slot="footer" class="dialog-footer">
		    	<!-- <el-button size='mini' @click="closeInnerModal">取 消</el-button> -->
		    	<el-button type="primary" size='mini' @click="saveOption">确 定</el-button>
		  	</span>
	    </el-dialog>
		  <span slot="footer" class="dialog-footer">
		    <el-button size='mini' @click="closeModal">取 消</el-button>
		    <el-button type="primary" size='mini' @click="saveOrUpdateQuestionNaire">确 定</el-button>
		  </span>
		</el-dialog>
		<!-- 预览 -->
		<el-dialog
		  :title='questionsDialog.content.name'
		  :visible.sync="questionsDialog.visible"
		  fullscreen
		  center
		  :before-close="closePreview">
		  <div style="margin-bottom:1em;color:teal">{{questionsDialog.content.description}}</div>
			<ul>
		  		<li class="question-list" v-for='(q,$index) in questionsDialog.content.questionVMs'>
					<div class="question-title">
						<span>{{$index+1}}. </span> {{q.name}}
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
				</li>
		  </ul>
		</el-dialog>
	</div>
</template>
<script>
	import getAxios from '@/http/getAxios'
	let axios = getAxios()
	export default {
		data(){
			return {
				questionNaires:[],
				questions:[],
				multipleSelection:[],
				loading:false,
				questionNaireDialog:{
					title:'',
					visible:false,
					form:{}
				},
				rules:{
					name:[{
						required: true, 
						message: '请输入栏目名称',
						trigger: 'blur' 
					}],
					description:[{
						required: true, 
						message: '请输入介绍信息',
						trigger: 'blur' 
					}]
				},
				choosedquestions:[],
				innerDialog:{
					visible:false
				},
				questionsDialog:{
					visible:false,
					content:{}
				}
			}
		},
		methods:{
			toPreview(row){
				axios.get('/questionnaire/findQuestionnaireVMById?id='+row.id)
				.then(({data:result})=>{
					this.questionsDialog.content = result.data;
					// console.log(result)
					this.questionsDialog.visible = true;
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
			},
			// 关闭预览
			closePreview(){
				this.questionsDialog.visible = false;
			},
			// 删除题目
			deleteQuestion(question){
				_.remove(this.choosedquestions,(item)=>{
					return item == question
				})
				this.choosedquestions.push({});
				this.choosedquestions.pop()
			},
			// 保存题目 
			saveOption(){
				// let ids = this.choosedquestions.map((item)=>{
				// 	return item.id
				// })
				// this.questionNaireDialog.form.questionIds = ids;
				this.closeInnerModal()
			},
			// 保存问卷
			saveOrUpdateQuestionNaire(){
				this.$refs.ruleForm.validate((valid)=>{
					if(valid){
						let ids = this.choosedquestions.map((item)=>{
							return item.id
						})
						this.questionNaireDialog.form.questionIds = ids;
						// console.log(this.questionNaireDialog.form)
						axios.post('/questionnaire/saveOrUpdateQuestionnaire',this.questionNaireDialog.form)
						.then(({data:result})=>{
							if(result.status == 200){
								this.$message({
		          		message: '保存成功',
		          		type: 'success'
		        		});
		        		this.findAllQuestionNaire();
		        		this.closeModal()
							}
						})
						.catch(()=>{
							this.$message.error('网络异常');
						})
					}
				})
			},
			// 修改问卷
			toUpdateQuestionNaire(row){
				this.questionNaireDialog.title = '修改问卷';
				axios.get('/questionnaire/findQuestionnaireVMById?id='+row.id)
				.then(({data:result})=>{
					this.choosedquestions = result.data.questionVMs;
					this.questionNaireDialog.form = row;
					this.questionNaireDialog.visible = true;
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
				
			},
			// 添加问卷
			toAddQuestionNaire(){
				this.questionNaireDialog.title = '添加问卷';
				this.questionNaireDialog.form = {};
				this.choosedquestions = [];
				this.questionNaireDialog.visible = true;
			},
			// 删除问卷
			deleteQuestionNaire(id){
				this.$confirm('此操作将永久删除该问卷, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        .then(()=>{
        	axios.get('/questionnaire/deleteQuestionnaireById?id='+id)
        	.then(()=>{
        		this.$message({
            	type: 'success',
            	message: '删除成功!'
          	});
          	this.findAllQuestionNaire();
        	})
        	.catch(()=>{
        		this.$message.error('网络异常');
        	})
        })
			},
			// 批量删除问卷
			batchDeleteQuestionNaire(){
				this.$confirm('此操作将永久删除该问卷, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        .then(()=>{
        	let ids = this.multipleSelection.map((item)=>{
        		return item.id
        	})
        	axios.post('/questionnaire/batchDeleteQuestion',{ids})
        	.then(()=>{
        		this.$message({
            	type: 'success',
            	message: '删除成功!'
          	});
          	this.findAllQuestionNaire();
        	})
        	.catch(()=>{
        		this.$message.error('网络异常');
        	})
        })
			},
			// 多选框
			handleSelectionChange(val) {
        this.multipleSelection = val;
      },
      // 加载所有的题目
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
      // 加载所有问卷
      findAllQuestionNaire(){
      	this.loading = true
      	axios.get('/questionnaire/findAllQuestionnaire')
				.then(({data:result})=>{
					this.questionNaires = result.data;
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
				.finally(()=>{
					this.loading = false
				})
      },
      // 关闭内层模态框
      closeInnerModal(){
      	this.innerDialog.visible = false;
      	// this.innerDialog.choosedquestions = [];
      	// this.questionNaireDialog.form.questionIds = [];
      },
      // 打开内层模态框
      openInnerModal(){
      	this.innerDialog.visible = true;
      },
      // 关闭模态框
      closeModal(){
      	this.questionNaireDialog.visible = false;
      }
		},
		created(){
			this.findAllQuestionNaire()
			this.findAllQuestion()
		}
	}
</script>
<style>
	.questionNaire {
		padding: 1em;
	}
	.questionNaire-btns {
		text-align: right;
		margin-bottom: 1em;
	}
	i.fa-eye {
		color: teal;
	}
	.question-list:first-child {
		border-top: 1px solid #ededed;
	}
</style>