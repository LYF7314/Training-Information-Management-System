<template>
    <div>
      <div class="container">
        <div class="handle-box">
          <el-input
            v-model="query.name"
            placeholder="学生姓名"
            class="handle-input mr10"
          ></el-input>
          <el-button type="primary" :icon="Search" @click="handleSearch">搜索</el-button>
          <el-button type="primary" :icon="Plus" @click="dialogFormVisible = true">新增</el-button>
        </div>

        <el-dialog v-model="dialogFormVisible" title="新增学员">
        <el-form :model="form">
          <el-form-item label="学员名称" :label-width="formLabelWidth">
            <el-input v-model="newStudent.studentName"/>
          </el-form-item>
          <el-form-item label="性别">
            <el-select v-model="newStudent.gender" placeholder="请选择学员性别">
              <el-option label="男" value="male" />
              <el-option label="女" value="female" />
            </el-select>
          </el-form-item>
          <el-form-item :label-width="formLabelWidth" label="公司">
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
          <el-form-item :label-width="formLabelWidth" label="邮箱">
            <el-input v-model.number="newStudent.email" />
          </el-form-item>
        </el-form>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="dialogFormVisible = false">取消</el-button>
            <el-button type="primary" @click="addStudent">
              新增
            </el-button>
          </span>
        </template>
      </el-dialog>

        <el-table
          :data="filteredData"
          border
          class="table"
          ref="multipleTable"
          header-cell-class-name="table-header"
        >
          <el-table-column
            prop="studentId"
            label="ID"
            width="55"
            align="center"
          ></el-table-column>
          <el-table-column prop="lesson" label="姓名"></el-table-column>
          <el-table-column prop="name" label="性别"></el-table-column>
          <el-table-column label="隶属公司"></el-table-column>
          <el-table-column prop="" label="职位"></el-table-column>
          <el-table-column prop="" label="业务水平"></el-table-column>
          <el-table-column prop="" label="邮箱地址"></el-table-column>
          <el-table-column label="操作" width="220">
            <template #default="scope">
              <el-button
                text
                :icon="Edit"
                @click="handleEdit(scope.row.studentId, scope.row)"
                v-permiss="15"
              >
                编辑
              </el-button>
              <el-button
                text
                :icon="Delete"
                class="red"
                @click="handleDelete(scope.row.courseId)"
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
      <el-form label-width="70px">
        <el-form-item label="职位">
          <el-input v-model.number="form.position"></el-input>
        </el-form-item>
        <el-form-item label="水平">
          <el-input v-model="form.level"></el-input>
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
    studentId: number;
    studentName: string;
    gender: string;
    company: string;
    position: string;
    level: string;
    email: string;
  }
  
  interface student {
    studentName: string;
    gender: string;
    company: string;
    position: string;
    level: string;
    email: string;
  }
  
  const query = reactive({
    type: 1,
    detail: "",
    name:"",
    pageIndex: 1,
    pageSize: 10,
  });
  const dialogFormVisible = ref(false);
  const formLabelWidth = "140px";
  const tableData = ref<TableItem[]>([]);
  const pageTotal = ref(0);
  const instance = getCurrentInstance();
  const newStudent: student = reactive({
    studentName: "",
    gender: "",
    company: "",
    position: "",
    level: "",
    email: ""
})
  // 获取表格数据
  const getData = () => {
    instance?.appContext.config.globalProperties.$http.get("/admin/student/list")
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

  const students:any = ref([]);

  const addStudent=()=>{
  instance?.appContext.config.globalProperties.$http.post("/admin/student/add",newStudent)
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
        if(!item.studentName)
          return false
        return item.studentName.indexOf(query.detail)!=-1
      })
      break;
    // case 2:
    //   filteredData.value = tableData.value.filter((item)=>{
    //     if(!item.courseTitle)
    //       return false
    //     return item.courseTitle.indexOf(query.detail)!=-1
    //   })
    //   break;
    // case 3:
    //   filteredData.value = tableData.value.filter((item)=>{
    //     if(!item.company)
    //       return false
    //     return item.company.indexOf(query.detail)!=-1
    //   })
    //   break;
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
            if(item.studentId == index){
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
    position: "",
    level: "",
  });
  let idx: number = -1;
  const handleEdit = (id: number, row: any) => {
    idx = id;
    form.position = row.cost;
    form.level = row.location;
    editVisible.value = true;
  };
  const saveEdit = () => {
    tableData.value.forEach((item,index)=>{
      if(item.studentId == idx){
          tableData.value[index].position = form.position;
          tableData.value[index].level = form.level;
          instance?.appContext.config.globalProperties.$http.post("/admin/student/update",tableData.value[index])
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
  