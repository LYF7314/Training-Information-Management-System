<template>
    <div class="container">
        <div class="form-box">
            <el-form ref="formRef" :rules="rules" :model="form" label-width="80px">
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
                    <el-button type="primary" @click="onSubmit(formRef)">表单提交</el-button>
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
const rules: FormRules = {
    name: [{ required: true, message: '请输入表单名称', trigger: 'blur' }],
};
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

// const course = reactive({
//     courseTitle: '',
//     startTime: '',
//     endTime: '',
//     location: '',
//     company: '',
//     cost: '',
// });

// const teacher = reactive({
//     teacherName: '',
//     professionalTitles: '',
//     specialization: '',
//     email: '',
//     phone: '',
// });

// 提交
const onSubmit = (formEl: FormInstance | undefined) => {
    // 表单校验
    if (!formEl) return;
    formEl.validate((valid) => {
        if (valid) {
            console.log(form);
            ElMessage.success('提交成功！');
        } else {
            return false;
        }
    });
};
// 重置
const onReset = (formEl: FormInstance | undefined) => {
    if (!formEl) return;
    formEl.resetFields();
};
</script>
