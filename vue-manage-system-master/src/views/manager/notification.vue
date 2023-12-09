<template>
    <div class="container">
        <div class="form-box">
            <el-form ref="formRef" :model="form" label-width="80px">
                <el-form-item label="课程名称" prop="name">
                    <el-input v-model="form.courseTitle"></el-input>
                </el-form-item>

                <el-form-item label="课程时间">
                    <el-date-picker
                    v-model="form.startTime"
                    type="datetime"
                    placeholder="请选择开始时间"
                    format="YYYY-MM-DD HH:mm:ss"
                    value-format="YYYY-MM-DD HH:mm:ss"
                    />
                    <span>&nbsp;&nbsp;</span>
                    <el-date-picker
                    v-model="form.endTime"
                    type="datetime"
                    placeholder="请选择结束时间"
                    format="YYYY-MM-DD HH:mm:ss"
                    value-format="YYYY-MM-DD HH:mm:ss"
                    />
                </el-form-item>

                <el-form-item label="课程地点">
                    <el-input v-model="form.location"></el-input>
                </el-form-item>

                <el-form-item label="开课公司">
                    <el-input v-model="form.company"></el-input>
                </el-form-item>

                <el-form-item label="课程学费">
                    <el-input v-model="form.cost"></el-input>
                </el-form-item>

                <el-form-item label="教师姓名">
                    <el-input v-model="form.teacherName"></el-input>
                </el-form-item>

                <el-form-item label="擅长领域">
                    <el-input v-model="form.specialization"></el-input>
                </el-form-item>
                
                <el-form-item label="教师职称" prop="region">
                    <el-select v-model="form.professionalTitles" placeholder="请选择">
                        <el-option key="Assistant" label="Assistant" value="Assistant"></el-option>
                        <el-option key="Lecturer" label="Lecturer" value="Lecturer"></el-option>
                        <el-option key="Reader" label="Reader" value="Reader"></el-option>
                        <el-option key="Professor" label="Professor" value="Professor"></el-option>
                    </el-select>
                </el-form-item>

                
                <el-form-item label="电子邮箱">
                    <el-input v-model="form.email"></el-input>
                </el-form-item>

                
                <el-form-item label="手机号码">
                    <el-input v-model="form.phone"></el-input>
                </el-form-item>

                <el-form-item>
                    <el-button type="primary" @click="addCourseInfo">表单提交</el-button>
                    <el-button @click="onReset(formRef)">重置表单</el-button>
                </el-form-item>
                
            </el-form>
        </div>
    </div>
</template>

<script setup lang="ts" name="baseform">
import { reactive, ref } from 'vue';
import { ElMessage } from 'element-plus';
import type { FormInstance, FormRules } from 'element-plus';
import { getCurrentInstance } from 'vue';

const formRef = ref<FormInstance>();
// interface course {
//     courseTitle: String,
//     startTime: String,
//     endTime: String,
//     location: String,
//     company: String,
//     cost: Number,
// }

// interface teacher {
//     teacherName: String,
//     professionalTitles: String,
//     specialization: String,
//     email: String,
//     phone: String,
// }

interface form {
  courseTitle: string;
  startTime: string;
  endTime: string;
  location: string;
  company: string;
  cost: string;
  teacherName: string,
  professionalTitles: string,
  specialization: string,
  email: string,
  phone: string
}

const form = reactive({
    courseTitle: '',
    startTime: '',
    endTime: '',
    location: '',
    company: '',
    cost: '',
    teacherName: '',
    professionalTitles: '',
    specialization: '',
    email: '',
    phone: '',
});

const transformedData = {
  teacher: {
    teacherName: form.teacherName,
    professionalTitles: form.professionalTitles,
    specialization: form.specialization,
    email: form.email,
    phone: form.phone
  },
  course: {
    courseTitle: form.courseTitle,
    startTime: form.startTime,
    endTime: form.endTime,
    location: form.location,
    company: form.company,
    cost: parseFloat(form.cost) // 将字符串转换为数字
  }
};

const instance = getCurrentInstance();

const addCourseInfo=()=>{
  instance?.appContext.config.globalProperties.$http.post("/student/info",transformedData)
  .then((res:any)=>{
    if(res.data.status===0){
      ElMessage.success("添加成功")
    }
    else{
      ElMessage.error(res.data.msg);
    }
  })
  .catch(()=>{
    ElMessage.error('服务器访问异常');
  })
}


// 重置
const onReset = (formEl: FormInstance | undefined) => {
    if (!formEl) return;
    formEl.resetFields();
};
</script>
