<template>
	<div>
		<el-row :gutter="20">
			<el-col :span="8">
				<el-card shadow="hover" class="mgb20" style="height: 252px">
					<div class="user-info">
						<el-avatar :size="120" :src="imgurl" />
						<div class="user-info-cont">
							<div class="user-info-name">{{ name }}</div>
							<div>{{ role }}</div>
						</div>
					</div>
					<div class="user-info-list">
						上次登录时间：
						<span>2022-10-01</span>
					</div>
					<div class="user-info-list">
						上次登录地点：
						<span>东莞</span>
					</div>
				</el-card>
				<el-card shadow="hover" style="height: 252px">
					<template #header>
						<div class="clearfix">
							<span>语言详情</span>
						</div>
					</template>
					Vue
					<el-progress :percentage="79.4" color="#42b983"></el-progress>
					TypeScript
					<el-progress :percentage="14" color="#f1e05a"></el-progress>
					CSS
					<el-progress :percentage="5.6"></el-progress>
					HTML
					<el-progress :percentage="1" color="#f56c6c"></el-progress>
				</el-card>
			</el-col>
			<el-col :span="16">
				<el-row :gutter="20" class="mgb20">
					<el-col :span="8">
						<el-card shadow="hover" :body-style="{ padding: '0px' }">
							<div class="grid-content grid-con-1">
								<el-icon class="grid-con-icon"><User /></el-icon>
								<div class="grid-cont-right">
									<div class="grid-num">{{ tab1.courseNumber }}</div>
									<div>经办课程数</div>
								</div>
							</div>
						</el-card>
					</el-col>
					<el-col :span="8">
						<el-card shadow="hover" :body-style="{ padding: '0px' }">
							<div class="grid-content grid-con-2">
								<el-icon class="grid-con-icon"><ChatDotRound /></el-icon>
								<div class="grid-cont-right">
									<div class="grid-num">{{ tab1.totalIncome }}</div>
									<div>总收入</div>
								</div>
							</div>
						</el-card>
					</el-col>
					<el-col :span="8">
						<el-card shadow="hover" :body-style="{ padding: '0px' }">
							<div class="grid-content grid-con-3">
								<el-icon class="grid-con-icon"><Goods /></el-icon>
								<div class="grid-cont-right">
									<div class="grid-num">{{ tab1.studentConversionRate }}</div>
									<div>学生转化率</div>
								</div>
							</div>
						</el-card>
					</el-col>
				</el-row>
				<el-card shadow="hover" style="height: 403px">
					<template #header>
						<div class="clearfix">
							<span>待办事项</span>
							<el-button style="float: right; padding: 3px 0" text>添加</el-button>
						</div>
					</template>

					<el-table :show-header="false" :data="todoList" style="width: 100%">
						<el-table-column width="40">
							<template #default="scope">
								<el-checkbox v-model="scope.row.status"></el-checkbox>
							</template>
						</el-table-column>
						<el-table-column>
							<template #default="scope">
								<div
									class="todo-item"
									:class="{
										'todo-item-del': scope.row.status
									}"
								>
									{{ scope.row.title }}
								</div>
							</template>
						</el-table-column>
					</el-table>
				</el-card>
			</el-col>
		</el-row>
		<el-row :gutter="20">
			<el-col :span="12">
				<el-card shadow="hover">
					<schart ref="bar" class="schart" canvasId="bar" :options="options"></schart>
				</el-card>
			</el-col>
			<el-col :span="6">
				<el-card shadow="hover">
					<schart ref="pie1" class="schart" canvasId="pie1" :options="options2"></schart>
				</el-card>
			</el-col>
			<el-col :span="6">
				<el-card shadow="hover">
					<schart ref="pie2" class="schart" canvasId="pie2" :options="options3"></schart>
				</el-card>
			</el-col>
		</el-row>
	</div>
</template>

<script setup lang="ts" name="dashboard">
import Schart from 'vue-schart';
import { reactive,ref,getCurrentInstance } from 'vue';
import imgurl from '../assets/img/img.jpg';
import { ElMessage } from 'element-plus';

const name = localStorage.getItem('ms_username');
const role: string = name === 'admin' ? '超级管理员' : '普通用户';

const options = reactive<any>({
	type: 'bar',
	title: {
		text: '课程情况图'
	},
	xRorate: 25,
	labels: [],
	datasets: []
});
const options2 = reactive<any>({
	type: 'pie',
	title: {
		text: '学生占比图'
	},
	labels: [],
	datasets: []
});
const options3 = reactive<any>({
	type: 'pie',
	title: {
		text: '讲师占比图'
	},
	labels: [],
	datasets: []
});
const todoList = reactive([
	{
		title: '今天要修复100个bug',
		status: false
	},
	{
		title: '今天要修复100个bug',
		status: false
	},
	{
		title: '今天要写100行代码加几个bug吧',
		status: false
	},
	{
		title: '今天要修复100个bug',
		status: false
	},
	{
		title: '今天要修复100个bug',
		status: true
	},
	{
		title: '今天要写100行代码加几个bug吧',
		status: true
	}
]);
const tab1 = ref<any>({});
const tab2 = ref<any>([]);
const tab3 = ref<any>([]);
const tab4 = ref<any>([]);
const instantce = getCurrentInstance()
const getData = () => {
	instantce?.appContext.config.globalProperties.$http.get('/report/admin').then((res: any) => {
		if(res.data.status==0){
			tab1.value = res.data.data;
		}
		else{
			ElMessage.error(res.data.msg);
		}
	}).catch(() => {
		ElMessage.error("服务器访问错误");
	});
	instantce?.appContext.config.globalProperties.$http.get('/report/course/barchart').then((res: any) => {
		if(res.data.status==0){
			tab2.value = res.data.data;
			tab2.value.splice(5)
			options.labels = tab2.value.map((item: any) => item.courseTitle)
			options.datasets = [];
			options.datasets.push({label:"学生数",data:tab2.value.map((item: any) => item.studentNumber)});
			options.datasets.push({label:"课程收入",data:tab2.value.map((item: any) => item. income)});
		}
		else{
			ElMessage.error(res.data.msg);
		}
	}).catch(() => {
		ElMessage.error("服务器访问错误");
	});
	instantce?.appContext.config.globalProperties.$http.get('/report/student/piechart').then((res: any) => {
		if(res.data.status==0){
			tab3.value = res.data.data;
			options2.labels = tab3.value.map((item: any) => item.level)
			options2.datasets.push({data:tab3.value.map((item: any) => item.studentNumber)})
		}
		else{
			ElMessage.error(res.data.msg);
		}
	}).catch(() => {
		ElMessage.error("服务器访问错误");
	});
	instantce?.appContext.config.globalProperties.$http.get('/report/teacher/piechart').then((res: any) => {
		if(res.data.status==0){
			tab4.value = res.data.data;
			options3.labels = tab4.value.map((item: any) => item.professionalTitles)
			options3.datasets.push({data:tab4.value.map((item: any) => item.teacherNumber)})
		}
		else{
			ElMessage.error(res.data.msg);
		}
	}).catch(() => {
		ElMessage.error("服务器访问错误");
	});
};
getData();
</script>

<style scoped>
.el-row {
	margin-bottom: 20px;
}

.grid-content {
	display: flex;
	align-items: center;
	height: 100px;
}

.grid-cont-right {
	flex: 1;
	text-align: center;
	font-size: 14px;
	color: #999;
}

.grid-num {
	font-size: 30px;
	font-weight: bold;
}

.grid-con-icon {
	font-size: 50px;
	width: 100px;
	height: 100px;
	text-align: center;
	line-height: 100px;
	color: #fff;
}

.grid-con-1 .grid-con-icon {
	background: rgb(45, 140, 240);
}

.grid-con-1 .grid-num {
	color: rgb(45, 140, 240);
}

.grid-con-2 .grid-con-icon {
	background: rgb(100, 213, 114);
}

.grid-con-2 .grid-num {
	color: rgb(100, 213, 114);
}

.grid-con-3 .grid-con-icon {
	background: rgb(242, 94, 67);
}

.grid-con-3 .grid-num {
	color: rgb(242, 94, 67);
}

.user-info {
	display: flex;
	align-items: center;
	padding-bottom: 20px;
	border-bottom: 2px solid #ccc;
	margin-bottom: 20px;
}

.user-info-cont {
	padding-left: 50px;
	flex: 1;
	font-size: 14px;
	color: #999;
}

.user-info-cont div:first-child {
	font-size: 30px;
	color: #222;
}

.user-info-list {
	font-size: 14px;
	color: #999;
	line-height: 25px;
}

.user-info-list span {
	margin-left: 70px;
}

.mgb20 {
	margin-bottom: 20px;
}

.todo-item {
	font-size: 14px;
}

.todo-item-del {
	text-decoration: line-through;
	color: #999;
}

.schart {
	width: 100%;
	height: 300px;
}
</style>
