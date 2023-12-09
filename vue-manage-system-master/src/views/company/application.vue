<template>
    <div class="container">
        <div class="form-box">
            <el-form ref="formRef" :model="form" label-width="80px">
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
                    <el-button type="primary" @click="addApplication">表单提交</el-button>
                    <!-- <el-button @click="">重置表单</el-button> -->
                </el-form-item>
                
            </el-form>
        </div>
    </div>
</template>

<script setup lang="ts" name="baseform">
import { ref, reactive,getCurrentInstance, computed} from "vue";
import { ElMessage } from 'element-plus';
import type { FormInstance, FormRules } from 'element-plus';
interface TableItem {
    costExpectation: number,
    companyName: string,
    course: string,
  }

  interface application{
    costExpectation: number,
    companyName: string,
    course: string,
  }

  const form = reactive({
    costExpectation: 0,
    companyName: '',
    course: '',
  })

const newApplication: application= reactive({
    costExpectation: 0,
    companyName: '',
    course: '',
});

const tableData = ref<TableItem[]>([]);
const instance = getCurrentInstance();

// 提交
const addApplication=()=>{
    instance?.appContext.config.globalProperties.$http.post("/student/info",newApplication)
    .then((res:any)=>{
        if(res.data.status===0){
            ElMessage.success("添加成功")
        }
        else {
            ElMessage.error(res.data.msg);
        }
    })
  .catch(()=>{
    ElMessage.error('服务器访问异常');
  })
}
</script>
