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
					<el-table-column width="200">
						<template #default="scope">
							<el-button :style="acceptButtonStyle(scope.row.accepted)" size="small" type="primary" @click="handleAccept(scope.row.messageId)">同意</el-button>
							<el-button size="small" @click="handleRead(scope.row.messageId)">标为已读</el-button>
						</template>
					</el-table-column>
				</el-table>
				<div class="handle-row">
					<el-button type="primary" @click="allRead">全部标为已读</el-button>
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
								<el-button :style="acceptButtonStyle(scope.row.accepted)" size="small" type="primary" @click="handleAccept(scope.row.messageId)">{{getButtonName(scope.row)}}</el-button>
								<el-button size="small" type="danger" @click="handleDel(scope.row.messageId)">删除</el-button>
							</template>
						</el-table-column>
					</el-table>
					<div class="handle-row">
						<el-button @click="allDelete" type="danger">删除全部</el-button>
					</div>
				</template>
			</el-tab-pane>
			<el-dialog title="消息详细" v-model="isDeatilShow" width="30%">
				<h2>{{ curMsg.title }}</h2>
				<p id="contentDetail">&nbsp;&nbsp;{{ curMsg.content }}</p>
				<!-- <template #footer>
					<span class="dialog-footer">
					<el-button @click="editVisible = false">取 消</el-button>
					<el-button type="primary" @click="saveEdit">确 定</el-button>
					</span>
				</template> -->
			</el-dialog>
			<!-- <el-tab-pane :label="`回收站(${state.recycle.length})`" name="third">
				<template v-if="message === 'third'">
					<el-table :data="state.recycle" :show-header="false" style="width: 100%">
						<el-table-column>
							<template #default="scope">
								<span class="message-title">{{ scope.row.title }}</span>
							</template>
						</el-table-column>
						<el-table-column prop="date" width="150"></el-table-column>
						<el-table-column width="120">
							<template #default="scope">
								<el-button @click="handleRestore(scope.$index)">还原</el-button>
							</template>
						</el-table-column>
					</el-table>
					<div class="handle-row">
						<el-button type="danger">清空回收站</el-button>
					</div>
				</template>
			</el-tab-pane> -->
		</el-tabs>
	</div>
</template>

<script setup lang="ts" name="tabs">
import { ElMessage } from 'element-plus';
import { ref, reactive,getCurrentInstance, computed } from 'vue';

const message = ref('first');
const state = reactive({
	unread: [
		{
			date: '2018-04-19 20:00:00',
			title: '【系统通知】该系统将于今晚凌晨2点到5点进行升级维护'
		},
		{
			date: '2018-04-19 21:00:00',
			title: '今晚12点整发大红包，先到先得'
		}
	],
	read: [
		{
			date: '2018-04-19 20:00:00',
			title: '【系统通知】该系统将于今晚凌晨2点到5点进行升级维护'
		}
	],
	recycle: [
		{
			date: '2018-04-19 20:00:00',
			title: '【系统通知】该系统将于今晚凌晨2点到5点进行升级维护'
		}
	]
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
		if(item.isRead){
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
	curMsg.value = messages.value.find((item:any)=>item.messageId === msgId);
}

const nums = ref<any>([]);

const allRead = ()=>{
	nums.value = []
	unreadMessages.value.forEach((item:any)=>{
		nums.value.push(item.messageId);
	})
	instance?.appContext.config.globalProperties.$http.post('/readList',{nums:nums.value})
	.then((res: any) => {
		if(res.data.status ===0){
			// 分拣消息
			unreadMessages.value.forEach((item:any)=>{
				item.isRead = true;
			});
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

const allDelete=()=>{
	nums.value = []
	readMessages.value.forEach((item:any)=>{
		nums.value.push(item.messageId);
	})
	instance?.appContext.config.globalProperties.$http.post('/message/deleteList',{nums:nums.value})
	.then((res: any) => {
		if(res.data.status ===0){
			// 分拣消息
			ElMessage.success("全部删除成功");
			getMessage();
		}
		else{
			ElMessage.error(res.data.msg);
		}
	})
	.catch(() => {
		ElMessage.error('服务器访问异常');
	});
}

const getButtonName = (msg:any)=>{
	switch (msg.type) {
		case 0:
			return '同意'
		case 1:
			return '缴费'
		case 2:
			return '签到'
		case 3:
			return '评教'
		default:
			break;
	}
}

const acceptButtonStyle = (isAccepted:boolean)=>{
	return {
		'visibility': isAccepted?'hidden':'visible',
	}
}

const handleRead = (msgId: number) => {
	instance?.appContext.config.globalProperties.$http.get(`/read?messageId=${msgId}`)
	.then((res:any)=>{
		if(res.data.status===0){
			messages.value.find((item:any)=>item.messageId === msgId).isRead = true;
			divideMessages();
		}
		else{
			ElMessage.error(res.data.msg);
		}
	})
	.catch(()=>{
		ElMessage.error('服务器访问异常');
	});
};
const handleAccept = (msgId: number)=>{
	const curMsg = messages.value.find((item:any)=>item.messageId === msgId);
	instance?.appContext.config.globalProperties.$http.get(`/admin/course/agree?studentId=${curMsg.senderId}&courseId=${curMsg.courseId}`)
	.then((res:any)=>{
		if(res.data.status===0){
			curMsg.accepted = true;
		}
		else{
			ElMessage.error(res.data.msg);
		}
	})
	.catch(()=>{
		ElMessage.error('服务器访问异常');
	});
}
const handleDel = (messageId: number) => {
	instance?.appContext.config.globalProperties.$http.post(`/message/delete`,{messageId})
	.then((res:any)=>{
		if(res.data.status===0){
			messages.value = messages.value.filter((item:any)=>item.messageId !== messageId);
			divideMessages();
		}
		else{
			ElMessage.error(res.data.msg);
		}
	})
	.catch(()=>{
		ElMessage.error('服务器访问异常');
	});
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
