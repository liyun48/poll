<template>
	<div class="school">
		<!-- 校园信息 -->
		<!-- 数据 -->
		<table class="school_table" v-loading="loading">
			<tr>
				<td>校园名称</td>
				<td>
					<input type="text" v-model='school.name'>
				</td>
			</tr>
			<tr>
				<td>校园介绍</td>
				<td>
					<input type="text" v-model='school.logopath'>
				</td>
			</tr>
			<tr>
				<td>校园地址</td>
				<td>
					<input type="text" v-model='school.address'>
				</td>
			</tr>
			<tr>
				<td>校园电话</td>
				<td>
					<input type="text" v-model='school.telephone'>
				</td>
			</tr>
			<tr>
				<td>版权信息</td>
				<td>
					<input type="text" v-model='school.copyright'>
				</td>
			</tr>
		</table>
		<!-- {{school}} -->
		<div class="school_btn">
			<el-button size='mini' @click='saveOrUpdateSchool'>保存</el-button>
		</div>
	</div>
</template>
<script>
	import getAxios from '@/http/getAxios'
	let axios = getAxios();
	export default {
		data(){
			return {
				school:{},
				loading:false
			}
		},
		methods:{
			// 保存
			saveOrUpdateSchool(){
				axios.post('/school/saveOrUpdate',this.school)
				.then(()=>{
					this.$message({
          				message: '保存成功',
          				type: 'success'
        			});
        			this.findSchool()
				})
				.catch((error)=>{
					this.$message({
          				message: '保存失败',
          				type: 'error'
        			});
				})
			},
			// 加载校园信息
			findSchool(){
				this.loading = true;
				axios.get('/school/findById?id=1')
				.then(({data:result})=>{
					this.school = result.data;
				})
				.catch(()=>{
					this.$message({
          				message: '加载失败',
          				type: 'error'
        			});
				})
				.finally(()=>{
					this.loading = false;
				})
			}
		},
		created(){
			this.findSchool()
		}
	}
</script>
<style>
	div {
		box-sizing: border-box;
	}
	.school_table {
		width: 98%;
		border-collapse: collapse;
		margin: 1em;
	}
	.school_table td {
		border: 1px solid #ededed;
		line-height: 30px;
	}
	.school_table td:first-child {
		width: 120px;
		text-align: center;
	}
	.school_table td input {
		border: none;
		line-height: 28px;
		font-size: 12px;
		color: #666;
		width: 98%;
		padding-left: 1em;
	}
	.school_btn {
		text-align: right;
		margin-right: 2em;
	}
</style>