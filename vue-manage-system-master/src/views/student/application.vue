<template>
    <div>
      <div class="container">
        <div class="handle-box">
          <el-input
            v-model="query.detail"
            placeholder="请输入内容"
            class="handle-input mr10"
          ></el-input>
          <el-button type="primary" :icon="Search" @click="handleSearch"
            >搜索</el-button
          >
        </div>
        <el-table
          :data="filteredData"
          border
          class="table"
          ref="multipleTable"
          header-cell-class-name="table-header"
        >
          <el-table-column
            prop="courseId"
            label="ID"
            width="55"
            align="center"
          ></el-table-column>
          <el-table-column prop="courseTitle" label="培训课程"></el-table-column>
          <el-table-column prop="teacherName" label="讲师名称"></el-table-column>
          <el-table-column prop="company" label="公司名称"></el-table-column>
          <el-table-column prop="cost" label="培训费用"></el-table-column>
          <el-table-column label="时间" width="320">
            <template #default="scope">
              {{ scope.row.startTime }} 至 {{ scope.row.endTime }}
            </template>
          </el-table-column>
              <el-table-column prop="location" label="地点"></el-table-column>
          <el-table-column label="操作" width="220" align="center">
            <template #default="scope">
              <el-button
                text
                icon="Check"
                @click="applyCourse(scope.row.courseId, scope.row)"
                v-permiss="15"
              >
                申请
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
          <el-form-item label="费用">
            <el-input v-model.number="form.cost"></el-input>
          </el-form-item>
          <el-form-item label="地址">
            <el-input v-model="form.location"></el-input>
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
  import { ref, reactive,getCurrentInstance, computed} from "vue";
  import { ElMessage, ElMessageBox } from "element-plus";
  import { Delete, Edit, Search, Plus } from "@element-plus/icons-vue";
  import { fetchData } from "../../api/index";
  
  interface TableItem {
    courseId: number,
    teacherId: number,
    courseTitle: string,
    startTime: string,
    endTime: string,
    location: string,
    company: string,
    cost: number,
    teacherName: string
  }
  interface course{
    teacherId: number,
    courseTitle: string,
    startTime: string,
    endTime: string,
    location: string,
    company: string,
    cost: number|null
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
  const newCourse: course = reactive({
    teacherId: 1,
    courseTitle: "",
    startTime: "",
    endTime: "",
    location: "",
    company: "",
    cost:null
  })
  // 获取表格数据
  const getData = () => {
    instance?.appContext.config.globalProperties.$http.get("/admin/course/list")
    .then((res:any)=>{
      if(res.data.status===0){
        filteredData.value = tableData.value = res.data.data
      }
      else{
        ElMessage.error(res.data.msg);
      }
    })
    .catch(()=>{
      ElMessage.error('服务器访问异常');
    })
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
  
  const beginAdd = ()=>{
    dialogFormVisible.value = true
    getTeachers()
  }
  
  const applyCourse=(courseId: number, row: any)=>{
    instance?.appContext.config.globalProperties.$http.post("/student/apply",courseId)
    .then((res:any)=>{
      if(res.data.status===0){
        ElMessage.success("发送申请成功")
        dialogFormVisible.value = false
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
          if(!item.courseTitle)
            return false
          return item.courseTitle.indexOf(query.detail)!=-1
        })
        break;
      case 3:
        filteredData.value = tableData.value.filter((item)=>{
          if(!item.company)
            return false
          return item.company.indexOf(query.detail)!=-1
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
        instance?.appContext.config.globalProperties.$http.post("/admin/course/delete",{
          courseId:index
        }).then((res:any)=>{
          if(res.data.status===0){
            ElMessage.success("删除成功");
            tableData.value.forEach((item,tindex)=>{
              if(item.courseId == index){
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
    cost: 0,
    location: "",
  });
  let idx: number = -1;
  const handleEdit = (id: number, row: any) => {
    idx = id;
    form.cost = row.cost;
    form.location = row.location;
    editVisible.value = true;
  };
  const saveEdit = () => {
    tableData.value.forEach((item,index)=>{
      if(item.courseId == idx){
          tableData.value[index].cost = form.cost;
          tableData.value[index].location = form.location;
          instance?.appContext.config.globalProperties.$http.post("/admin/course/update",tableData.value[index])
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
  