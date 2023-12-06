<template>
    <div class="container">
        <div class="form-box">
            <el-form ref="formRef" :rules="rules" :model="form" label-width="80px">
                <el-form-item label="公司名称" prop="name">
                    <el-input v-model="form.companyName"></el-input>
                </el-form-item>

                <el-form-item label="课程预期" prop="desc">
                    <el-input type="textarea" rows="5" v-model="form.course"></el-input>
                </el-form-item>

                
                <el-form-item label="学费预期" prop="name">
                    <el-input v-model="form.costExpectation"></el-input>
                </el-form-item>

                <el-form-item>
                    <el-button type="primary" @click="">表单提交</el-button>
                    <el-button @click="">重置表单</el-button>
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

const form = reactive({
    costExpectation: '',
    companyName: '',
    course: '',
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
