<template>
	<div class="surveyInfo">
		<!-- 课调创建 -->
		<!-- 按钮区 -->
		<div class="btns">
			<el-button size='mini' @click='toAddSurveyInfo'>添加</el-button>
			<el-button size='mini' @click='batchDeleteSurveyInfo'>批量删除</el-button>
		</div>
		<!-- 内容区 -->
		<div class="table">
			<el-table
		    :data="surveyInfo"
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
		      prop="clazzVM.grade.name"
		      label="年级"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="clazzVM.name"
		      label="班级"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="course.name"
		      label="课程"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="qnVM.name"
		      label="问卷"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="user.name"
		      label="讲师"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      prop="surveydate"
		      label="课调时间"
		      align='center'>
		    </el-table-column>
		    <el-table-column
		      label="操作"
		      width='100'
		      align='center'>
		      <template slot-scope='{row}'>
		      	<i class="fa fa-eye" title="预览" @click='toPreview(row)'></i>
		      	<i class="fa fa-pencil" title="修改" @click='toUpdateSurveyInfo(row)'></i>
						<i class="fa fa-trash" title="删除" @click='deleteSurveyInfo(row.id)'></i>
		      </template>
		    </el-table-column>
		  </el-table>
		</div>
		<!-- 模态框 -->
		<el-dialog
		  :title='surveyInfoDialog.title'
		  :visible.sync="surveyInfoDialog.visible"
		  width="50%"
		  :before-close="closeModal">
		  <!-- {{surveyInfoDialog.form}} -->
		  <el-form :model="surveyInfoDialog.form" size='mini' label-position='left' label-width="6em" :rules="rules" ref="ruleForm">
				<el-form-item label="年级" prop=''>
			    <el-select v-model="surveyInfoDialog.form.gradeId" placeholder="请选择年级" style='width:70%'>
			      <el-option :key="g.id" :label="g.name" :value='g.id' v-for='g in grades'></el-option>
			    </el-select>
			  </el-form-item>
			  <el-form-item label="班级" prop=''>
			    <el-select v-model="surveyInfoDialog.form.clazzId" placeholder="请选择班级" style='width:70%'>
			      <el-option :key="c.id" :label="c.name" :value='c.id' v-for='c in clazz'></el-option>
			    </el-select>
			  </el-form-item>
			  <el-form-item label="课程" prop=''>
			    <el-select v-model="surveyInfoDialog.form.courseId" placeholder="请选择课程" style='width:70%'>
			      <el-option :key="c.id" :label="c.name" :value='c.id' v-for='c in courses'></el-option>
			    </el-select>
			  </el-form-item>
			  <el-form-item label="问卷" prop=''>
			    <el-select v-model="surveyInfoDialog.form.questionnaireId" placeholder="请选择问卷" style='width:70%'>
			      <el-option :key="q.id" :label="q.name" :value='q.id' v-for='q in questionNaires'></el-option>
			    </el-select>
			  </el-form-item>
			  <el-form-item label="讲师" prop=''>
			    <el-select v-model="surveyInfoDialog.form.userId" placeholder="请选择问卷" style='width:70%'>
			      <el-option :key="u.id" :label="u.name" :value='u.id' v-for='u in users'></el-option>
			    </el-select>
			  </el-form-item>
			  <el-form-item label="开启" prop=''>
			  	<el-checkbox v-model="surveyInfoDialog.form.status" true-label='已审核' false-label='未审核'>创建完成后开启课调</el-checkbox>
			  </el-form-item>
		  </el-form>
		  <span slot="footer" class="dialog-footer">
		    <el-button size='mini' @click="closeModal">取 消</el-button>
		    <el-button type="primary" size='mini' @click="saveOrUpdateSurveyInfo">确 定</el-button>
		  </span>
		</el-dialog>
		<!-- 预览 -->
		<el-dialog
		  :title='surveyInfoPreview.title'
		  :visible.sync="surveyInfoPreview.visible"
		  fullscreen
		  center
		  :before-close="closePreview">
		  <ul class="surveyInfoPreview_content">
		  	<li>班级：{{surveyInfoPreview.content.clazzVM.name}}</li>
		  	<li>讲师：{{surveyInfoPreview.content.user.name}}</li>
		  	<li>时间：{{surveyInfoPreview.content.surveydate}}</li>
		  	<li>课程：{{surveyInfoPreview.content.course.name}}</li>
		  </ul>
			<ul>
		  	<li class="question-list" v-for='(q,$index) in surveyInfoPreview.content.qnVM.questionVMs'>
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
				surveyInfo:[],
				grades:[],
				clazz:[],
				courses:[],
				questionNaires:[],
				users:[],
				multipleSelection:[],
				loading:false,
				rules:{},
				surveyInfoDialog:{
					title:'',
					visible:false,
					form:{}
				},
				surveyInfoPreview:{
					title:'',
					visible:false,
					content:{
						course:{},
						clazzVM:{
							grade:{}
						},
						user:{},
						qnVM:{
							questionVMs:[]
						}
					}
				}
			}
		},
		methods:{
			// 关闭预览
			closePreview(){
				this.surveyInfoPreview.visible = false
			},
			// 预览
			toPreview(row){
				this.surveyInfoPreview.title = row.qnVM.name;
				// axios.get('/survey/findSurveyVMById?id='+id)
				// .then(({data:result})=>{
				// 	this.surveyInfo.content = result.data;
				// })
				// .catch(()=>{
				// 	this.$message.error('网络异常');
				// })
				this.surveyInfoPreview.content = row;
				// console.log(this.surveyInfoPreview.content)
				this.surveyInfoPreview.visible = true;
			},
			// 保存课调信息
			saveOrUpdateSurveyInfo(){
				axios.post('/survey/saveOrUpdateSurvey',this.surveyInfoDialog.form)
				.then(({data:result})=>{
					if(result.status == 200){
						this.$message({
		        message: '保存成功',
		        type: 'success'
		      });
						this.findAllSurveyInfo();
		      this.closeModal();
					}
		      
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
			},
			// 添加课调信息
			toAddSurveyInfo(){
				this.surveyInfoDialog.title = '添加课调信息'
				this.surveyInfoDialog.form = {}
				this.surveyInfoDialog.visible = true;
			},
			// 修改课调信息
			toUpdateSurveyInfo(row){
				this.surveyInfoDialog.title = '修改课调信息'
				let surveyInfo = _.clone(row)
				surveyInfo.courseId = surveyInfo.course.id;
				delete surveyInfo.course
				surveyInfo.clazzId = surveyInfo.clazzVM.id;
				surveyInfo.gradeId = surveyInfo.clazzVM.grade.id;
				delete surveyInfo.clazzVM
				surveyInfo.questionnaireId = surveyInfo.qnVM.id
				delete surveyInfo.qnVM
				surveyInfo.userId = surveyInfo.user.id
				delete surveyInfo.user
				delete surveyInfo.charge
				this.surveyInfoDialog.form = surveyInfo;
				this.surveyInfoDialog.visible = true;
			},
			// 批量删除课调信息
			batchDeleteSurveyInfo(){
				this.$confirm('此操作将永久删除该题目, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        .then(()=>{
        	let ids = this.multipleSelection.map((item)=>{
        		return item.id
        	})
        	axios.post('/survey/batchDeleteSurveyById',{ids})
        	.then(()=>{
        		this.$message({
            	type: 'success',
            	message: '删除成功!'
            });
          	this.findAllSurveyInfo();
        	})
        	.catch(()=>{
        		this.$message.error('网络异常');
        	})
        })
			},
			// 删除课调信息
			deleteSurveyInfo(id){
				this.$confirm('此操作将永久删除该题目, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        .then(()=>{
        	axios.get('/survey/deleteSurveyById',{
        		params:{id}
        	})
        	.then(()=>{
        		this.$message({
            	type: 'success',
            	message: '删除成功!'
            });
          	this.findAllSurveyInfo();
        	})
        	.catch(()=>{
        		this.$message.error('网络异常');
        	})
        })
			},
			// 加载年级信息
			findAllGrade(){
				axios.get('/grade/findAll')
				.then(({data:result})=>{
					this.grades = result.data
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
			},
			// 加载班级信息
			findAllClazz(){
				axios.get('/clazz/findAll')
				.then(({data:result})=>{
					this.clazz = result.data
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
			},
			// 加载课程信息
			findAllCourse(){
				axios.get('/course/findAllCourse')
				.then(({data:result})=>{
					this.courses = result.data
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
			},
			// 加载问卷信息
			findAllQuestionNaire(){
				axios.get('/questionnaire/findAllQuestionnaire')
				.then(({data:result})=>{
					this.questionNaires = result.data
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
			// 加载课调信息
			findAllSurveyInfo(){
				this.loading = true;
				axios.get('/survey/findAllSurveyVM')
				.then(({data:result})=>{
					this.surveyInfo = result.data;
				})
				.catch(()=>{
					this.$message.error('网络异常');
				})
				.finally(()=>{
					this.loading = false;
				})
			},
			// 多选框
			handleSelectionChange(val){
				this.multipleSelection = val
			},
			// 关闭模态框
			closeModal(){
				this.surveyInfoDialog.visible = false;
			}
		},
		created(){
			this.findAllSurveyInfo();
			this.findAllGrade();
			this.findAllClazz();
			this.findAllCourse();
			this.findAllQuestionNaire();
			this.findAllUser();
		}
	}
</script>
<style>
	.surveyInfo {
		padding: 1em;
	}
	.btns {
		text-align: right;
		margin-bottom: 1em;
	}
	i.fa {
		margin: 0 .3em;
		cursor: pointer;
	}
	i.fa-pencil {
		color: #409EFF;
	}
	i.fa-trash {
		color: #F56C6C
	}
	i.fa-eye {
		color: teal;
	}
	.surveyInfoPreview_content::after {
		content:"";
		display: block;
		clear: both;
	}
	.surveyInfoPreview_content>li {
		float: left;
		text-align: center;
		line-height: 3em;
		width: 24.9%;
		margin-bottom: 1.5em;
		border-right: 1px solid #ededed;
		overflow: hidden;
	}
	.surveyInfoPreview_content>li:last-child {
		border-right: none;
	}
	.question-list{
		padding: 1em 0;
		border-bottom: 1px solid #ededed;
		padding-left: 1em;
	}
	.question-list:first-child {
		border-top: 1px solid #ededed;
	}
</style>