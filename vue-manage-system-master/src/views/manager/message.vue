<template>
	<div class="container">
		<el-tabs v-model="message">

			<el-tab-pane :label="`未读消息(${unreadMessages.length})`" name="first">
				<el-table :data="unreadMessages" :show-header="false" style="width: 100%">
					<el-table-column>
						<template #default="scope">
							<span @click="readDetail(scope.row.messageId)" class="message-title">{{ scope.row.title }}</span>
						</template>
					</el-table-column>
					<el-table-column prop="sendTime" width="180"></el-table-column>
					<el-table-column width="100">
						<template #default="scope">
							<!-- <el-button size="small" type="primary" @click="">同意</el-button> -->
							<el-button size="small" @click="handleRead(scope.$index)">标为已读</el-button>
						</template>
					</el-table-column>
				</el-table>
				<div class="handle-row">
					<el-button type="primary">全部标为已读</el-button>
				</div>
			</el-tab-pane>

			<el-tab-pane :label="`已读消息(${readMessages.length})`" name="second">
				<template v-if="message === 'second'">
					<el-table :data="readMessages" :show-header="false" style="width: 100%">
						<el-table-column>
							<template #default="scope">
								<span @click="readDetail(scope.row.messageId)" class="message-title">{{ scope.row.title }}</span>
							</template>
						</el-table-column>
						<el-table-column prop="sendTime" width="160"></el-table-column>
						<el-table-column width="150">
							<template #default="scope">
								<el-button size="small" type="primary" @click="handleDel(scope.$index)">同意</el-button>
								<el-button size="small" type="danger" @click="handleDel(scope.$index)">删除</el-button>
							</template>
						</el-table-column>
					</el-table>
					<div class="handle-row">
						<el-button type="danger">删除全部</el-button>
					</div>
				</template>
			</el-tab-pane>

			<el-dialog title="消息详细" v-model="isDeatilShow" width="30%">
				<h2>{{ curMsg.title }}</h2>
				<p id="contentDetail">&nbsp;&nbsp;{{ curMsg.content }}</p>
			</el-dialog>

		</el-tabs>
	</div>
</template>

<script setup lang="ts" name="tabs">
import { ElMessage } from 'element-plus';
import { ref, reactive,getCurrentInstance } from 'vue';

// const message = ref('first');
// const state = reactive({
// 	unread: [
// 		{
// 			date: '2018-04-19 20:00:00',
// 			title: '【系统通知】该系统将于今晚凌晨2点到5点进行升级维护'
// 		},
// 		{
// 			date: '2018-04-19 21:00:00',
// 			title: '今晚12点整发大红包，先到先得'
// 		}
// 	],
// 	read: [
// 		{
// 			date: '2018-04-19 20:00:00',
// 			title: '【系统通知】该系统将于今晚凌晨2点到5点进行升级维护'
// 		}
// 	],
// 	recycle: [
// 		{
// 			date: '2018-04-19 20:00:00',
// 			title: '【系统通知】该系统将于今晚凌晨2点到5点进行升级维护'
// 		}
// 	]
// });

const state = reactive({
  unread: [],
  read: [],
  recycle: []
});

const messages = ref<any[]>([]);
const readMessages = ref<any[]>([]);
const unreadMessages = ref<any[]>([]);

const instance = getCurrentInstance();

const getMessage = ()=>{
	instance?.appContext.config.globalProperties.$http.get('/messageList')
	.then((res: any) => {
		if(res.data.status ===0){
			// 分拣消息
			messages.value = res.data.data;
			divideMessages();
		}
		else{
			ElMessage.error(res.data.msg);
		}
	})
	.catch(() => {
		ElMessage.error('服务器访问异常');
	});
}
getMessage();

const divideMessages = ()=>{
	unreadMessages.value = [];
	readMessages.value = [];
	messages.value.forEach((item: any) => {
		if(item.read){
			readMessages.value.push(item);
		}
		else{
			unreadMessages.value.push(item);
		}
	});
}

const isDeatilShow = ref(false);
const curMsgId = ref(-1)
const curMsg = ref<any>({})
const readDetail=(msgId:number)=>{
	isDeatilShow.value = true;
	curMsgId.value = msgId;
    curMsg.button = "OK"
	curMsg.value = messages.value.find((item:any)=>item.messageId === msgId);
}

const handleRead = (index: number) => {
	const item = state.unread.splice(index, 1);
	state.read = item.concat(state.read);
};
const handleDel = (index: number) => {
	const item = state.read.splice(index, 1);
	state.recycle = item.concat(state.recycle);
};
const handleRestore = (index: number) => {
	const item = state.recycle.splice(index, 1);
	state.read = item.concat(state.read);
};
</script>

<style>
.message-title {
	cursor: pointer;
}
.handle-row {
	margin-top: 30px;
}
#contentDetail{
	margin-top: 30px;
}
</style>
