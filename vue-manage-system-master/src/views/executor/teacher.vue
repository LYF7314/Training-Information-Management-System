<template>
    <div>
      <div class="container">
        <div class="handle-box">
          <el-input
            v-model="query.detail"
            placeholder="教师姓名"
            class="handle-input mr10"
          ></el-input>
          <el-button type="primary" :icon="Search" @click="handleSearch">搜索</el-button>
          <el-button type="primary" :icon="Plus" @click="dialogFormVisible = true">新增</el-button>
        </div>

        <el-dialog v-model="dialogFormVisible" title="新增教师">
          <el-form :model="form">
          <el-form-item label="教师名称" :label-width="formLabelWidth">
            <el-input v-model="newTeacher.teacherName"/>
          </el-form-item>
          <el-form-item label="职位" :label-width="formLabelWidth">
            <el-select v-model="newTeacher.professionalTitles" placeholder="请选择学员职位">
              <el-option label="Assistant" value="Assistant" />
              <el-option label="Lecturer" value="Lecturer" />
              <el-option label="Reader" value="Reader" />
              <el-option label="Professor" value="Professor" />
            </el-select>
          </el-form-item>
          <el-form-item :label-width="formLabelWidth" label="擅长领域">
            <el-input v-model="newTeacher.specialization"/>
          </el-form-item>
          <el-form-item :label-width="formLabelWidth" label="邮箱">
            <el-input v-model="newTeacher.email" />
          </el-form-item>
          <el-form-item :label-width="formLabelWidth" label="电话">
            <el-input v-model="newTeacher.phone" />
          </el-form-item>
          </el-form>
          <template #footer>
            <span class="dialog-footer">
              <el-button @click="dialogFormVisible = false">取消</el-button>
              <el-button type="primary" @click="addTeacher">
                新增
              </el-button>
            </span>
          </template>
        </el-dialog>

        <el-table
          :data="tableData"
          border
          class="table"
          ref="multipleTable"
          header-cell-class-name="table-header"
        >
          <el-table-column
            prop="teacherId"
            label="ID"
            width="55"
            align="center"
          ></el-table-column>
          <el-table-column prop="teacherName" label="姓名"></el-table-column>
          <el-table-column prop="specialization" label="专业领域"></el-table-column>
          <el-table-column prop="professionalTitles" label="职称"></el-table-column>
          <el-table-column prop="email" label="邮箱地址"></el-table-column>
          <el-table-column prop="phone" label="电话"></el-table-column>
          <el-table-column label="操作" width="220" align="center">
            <template #default="scope">
              <el-button
                text
                :icon="Edit"
                @click="handleEdit(scope.row.teacherId, scope.row)"
                v-permiss="15"
              >
                编辑
              </el-button>
              <el-button
                text
                :icon="Delete"
                class="red"
                @click="handleDelete(scope.row.teacherId)"
                v-permiss="16"
              >
                删除
              </el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="pagination">
          <el-pagination
            background
            layout="total, prev, pager, next"
            :current-page="query.pageIndex"
            :page-size="query.pageSize"
            :total="pageTotal"
            @current-change="handlePageChange"
          ></el-pagination>
        </div>
      </div>
  
      <!-- 编辑弹出框 -->
      <el-dialog title="编辑" v-model="editVisible" width="30%">
        <el-form :model="form">
          <el-form-item label="教师名称" :label-width="formLabelWidth">
            <el-input v-model="form.teacherName"/>
          </el-form-item>
          <el-form-item label="职位" :label-width="formLabelWidth">
            <el-select v-model="form.professionalTitles" placeholder="请选择学员职位">
              <el-option label="Assistant" value="Assistant" />
              <el-option label="Lecturer" value="Lecturer" />
              <el-option label="Reader" value="Reader" />
              <el-option label="Professor" value="Professor" />
            </el-select>
          </el-form-item>
          <el-form-item :label-width="formLabelWidth" label="擅长领域">
            <el-input v-model="form.specialization"/>
          </el-form-item>
          <el-form-item :label-width="formLabelWidth" label="邮箱">
            <el-input v-model="form.email" />
          </el-form-item>
          <el-form-item :label-width="formLabelWidth" label="电话">
            <el-input v-model="form.phone" />
          </el-form-item>
        </el-form>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="editVisible = false">取 消</el-button>
            <el-button type="primary" @click="saveEdit">确 定</el-button>
          </span>
        </template>
      </el-dialog>
    </div>
  </template>
  
  <script setup lang="ts" name="basetable">
  import { ref, reactive } from "vue";
  import { ElMessage, ElMessageBox } from "element-plus";
  import { Delete, Edit, Search, Plus } from "@element-plus/icons-vue";
  import { fetchData } from "../../api/index";
  import { getCurrentInstance } from 'vue';
  
  interface TableItem {
    teacherId:number;
    teacherName: string;
    professionalTitles: string;
    specialization: string;
    email: string;
    phone: string;
  }

  interface teacher {
    teacherName: string;
    professionalTitles: string;
    specialization: string;
    email: string;
    phone: string;
  }
  
  const query = reactive({
    type: 1,
    detail: "",
    pageIndex: 1,
    pageSize: 10,
  });
  const dialogFormVisible = ref(false);
  const formLabelWidth = "140px";
  const tableData = ref<TableItem[]>([]);
  const pageTotal = ref(0);
  const instance = getCurrentInstance();
  const newTeacher: teacher = reactive({
    teacherName: "",
    professionalTitles: "",
    specialization: "",
    email: "",
    phone: ""
  })
  // 获取表格数据
  const getData = () => {
    instance?.appContext.config.globalProperties.$http.get("/admin/teacher/list")
    .then((res:any) => {
      if(res.data.status===0){
      filteredData.value = tableData.value = res.data.data
      }
      else{
        ElMessage.error(res.data.msg);
      }
    });
    fetchData().then((res) => {
      tableData.value = res.data.list;
      pageTotal.value = res.data.pageTotal || 50;
    });
  };
  getData();
  
  const teachers:any = ref([]);

  const getTeachers = ()=>{
  instance?.appContext.config.globalProperties.$http.get("/admin/teacher/list")
  .then((res:any)=>{
    if(res.data.status===0){
      teachers.value = res.data.data
    }
    else{
      ElMessage.error(res.data.msg);
    }
  })
  .catch(()=>{
    ElMessage.error('服务器访问异常');
  })
}

const addTeacher=()=>{
  instance?.appContext.config.globalProperties.$http.post("/admin/teacher/add",newTeacher)
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

  // 过滤查询
  const filteredData = ref<TableItem[]>([]);

  // 查询操作
  const handleSearch = () => {
  // query.pageIndex = 1;
  if(query.detail==="") {
    filteredData.value = tableData.value
    return
  }
  switch(query.type){
    case 1:
      filteredData.value = tableData.value.filter((item)=>{
        if(!item.teacherName)
          return false
        return item.teacherName.indexOf(query.detail)!=-1
      })
      break;
    case 2:
      filteredData.value = tableData.value.filter((item)=>{
        if(!item.email)
          return false
        return item.email.indexOf(query.detail)!=-1
      })
      break;
    case 3:
      filteredData.value = tableData.value.filter((item)=>{
        if(!item.phone)
          return false
        return item.phone.indexOf(query.detail)!=-1
      })
      break;
  }
};
  // 分页导航
  const handlePageChange = (val: number) => {
    query.pageIndex = val;
    getData();
  };
  
  // 删除操作
  const handleDelete = (index: number) => {
    // 二次确认删除
    ElMessageBox.confirm("确定要删除吗？", "提示", {
    type: "warning",
  })
    .then(() => {
      instance?.appContext.config.globalProperties.$http.post("/admin/student/delete",{
        courseId:index
      }).then((res:any)=>{
        if(res.data.status===0){
          ElMessage.success("删除成功");
          tableData.value.forEach((item,tindex)=>{
            if(item.teacherId == index){
              tableData.value.splice(tindex, 1);
            }
          })
        }
      }).catch((res:any)=>{
        ElMessage.error(res.data.msg);
      })
    })
    .catch(() => {});
  };
  
  // 表格编辑时弹窗和保存
  const editVisible = ref(false);
  let form = reactive({
    teacherName: "",
    professionalTitles: "",
    specialization: "",
    email: "",
    phone: ""
  });
  let idx: number = -1;
  const handleEdit = (index: number, row: any) => {
    idx = index;
    form.teacherName = row.teacherName;
    form.professionalTitles = row.professionalTitles;
    form.specialization = row.specialization;
    form.email = row.email;
    form.phone = row.phone;
    editVisible.value = true;
  };
  const saveEdit = () => {
    tableData.value.forEach((item,index)=>{
      if(item.teacherId == idx){
          tableData.value[index].teacherName = form.teacherName;
          tableData.value[index].professionalTitles = form.professionalTitles;
          tableData.value[index].specialization = form.specialization;
          tableData.value[index].email = form.email;
          tableData.value[index].phone = form.phone;
          instance?.appContext.config.globalProperties.$http.post("/admin/teacher/update",tableData.value[index])
          .then((res:any)=>{
            if(res.data.status===0){
              ElMessage.success("修改成功");
              editVisible.value = false;
              handleSearch();
            }
            else{
              ElMessage.error(res.data.msg);
            }
          })
          .catch(()=>{
            ElMessage.error('服务器访问异常');
          })
        }
    })
    
  };
  </script>
  
  <style scoped>
  .handle-box {
    margin-bottom: 20px;
  }
  
  .handle-select {
    width: 120px;
  }
  
  .handle-input {
    width: 300px;
  }
  .table {
    width: 100%;
    font-size: 14px;
  }
  .red {
    color: #f56c6c;
  }
  .mr10 {
    margin-right: 10px;
  }
  .table-td-thumb {
    display: block;
    margin: auto;
    width: 40px;
    height: 40px;
  }
  </style>
  