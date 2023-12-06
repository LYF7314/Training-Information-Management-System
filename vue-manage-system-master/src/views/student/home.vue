<template>
  <el-card class="box-card">
    <el-descriptions
        title="学生主页"
        column="7"
        size="large"
        direction="vertical"
    >
        <el-descriptions-item label="姓名">{{ studentInfo.studentName }}</el-descriptions-item>
        <el-descriptions-item label="性别">{{ studentInfo.gender }}</el-descriptions-item>
        <el-descriptions-item label="company" :span="2">{{ studentInfo.company }}</el-descriptions-item>
        <el-descriptions-item label="工作岗位">
          <el-tag size="large">{{ studentInfo.position }}</el-tag>
        </el-descriptions-item>
        <el-descriptions-item label="技术水平">
          <el-tag size="large">{{ studentInfo.level }}</el-tag>
        </el-descriptions-item>
        <el-descriptions-item label="邮件地址">{{ studentInfo.email }}</el-descriptions-item>
    </el-descriptions>

    <el-button @click="dialogFormVisible = true" type="primary" round>
      提交个人信息
    </el-button>

  </el-card>

  <el-dialog v-model="dialogFormVisible" title="新增学员">

    <el-form :model="form">
      <el-form-item label="名称">
        <el-input v-model="newStudent.studentName"/>
      </el-form-item>

      <el-form-item label="性别">
        <el-select v-model="newStudent.gender" placeholder="请选择学员性别">
          <el-option label="男" value="male" />
          <el-option label="女" value="female" />
        </el-select>
      </el-form-item>

      <el-form-item label="公司">
        <el-input v-model="newStudent.company"/>
      </el-form-item>

      <el-form-item label="职位">
        <el-select v-model="newStudent.position" placeholder="请选择学员职位">
          <el-option label="Product" value="Product" />
          <el-option label="Frontend" value="Frontend" />
          <el-option label="Backend" value="Backend" />
          <el-option label="FullStack" value="FullStack" />
          <el-option label="Architect" value="Architect" />
          <el-option label="Planner" value="Planner" />
        </el-select>
      </el-form-item>

      <el-form-item label="水平">
        <el-select v-model="newStudent.level" placeholder="请选择学员水平">
          <el-option label="Junior" value="Junior" />
          <el-option label="Intermediate" value="Intermediate" />
          <el-option label="Senior" value="Senior" />
          <el-option label="Lead" value="Lead" />
          <el-option label="Architect" value="Architect" />
          <el-option label="Principal" value="Principal" />
        </el-select>
      </el-form-item>

      <el-form-item label="邮箱">
        <el-input v-model.number="newStudent.email" />
      </el-form-item>

    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取消</el-button>
        <el-button type="primary" @click="addStudentInfo">
          保存
        </el-button>
      </span>
    </template>
  </el-dialog>

</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import { defineComponent } from 'vue';
import { getCurrentInstance } from 'vue';
import { ElMessage, ElMessageBox } from "element-plus";
import { fetchData } from "../../api/index";

const dialogTableVisible = ref(false)
const dialogFormVisible = ref(false)
const formLabelWidth = '140px'


interface student {
  studentName: string;
  gender: string;
  company: string;
  position: string;
  level: string;
  email: string;
}

const form = reactive({
  studentName: '',
  region: '',
  date1: '',
  date2: '',
  delivery: false,
  type: [],
  resource: '',
  desc: '',
})

const studentInfo: student = reactive({
  studentName: "无",
  gender: "无",
  company: "无",
  position: "未上传",
  level: "未上传",
  email: "无",
})

const newStudent: student = reactive({
    studentName: "",
    gender: "",
    company: "",
    position: "",
    level: "",
    email: ""
})

const instance = getCurrentInstance();
const tableData = ref<student[]>([]);

const getData = () => {
  instance?.appContext.config.globalProperties.$http.get("/admin/student/list")
  .then((res:any) => {
    if(res.data.status===0){
      tableData.value = res.data.data
    }
    else{
      ElMessage.error(res.data.msg);
    }
  });
  fetchData().then((res) => {
    tableData.value = res.data.list;
  });
};
getData();

const addStudentInfo=()=>{
  instance?.appContext.config.globalProperties.$http.post("/student/info",newStudent)
  .then((res:any)=>{
    if(res.data.status===0){
      ElMessage.success("添加成功")
      dialogFormVisible.value = false
      getData()
    }
    else{
      ElMessage.error(res.data.msg);
    }
  })
  .catch(()=>{
    ElMessage.error('服务器访问异常');
  })
}
</script>

<style scoped>
.el-descriptions {
  margin-top: 20px;
}
</style>
