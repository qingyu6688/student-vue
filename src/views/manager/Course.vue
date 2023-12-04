<template>
<div>
  <div class="card" style="margin-bottom: 10px">
    <el-input style="width: 260px;margin-right: 10px" v-model="data.name" placeholder="请输入课程名称查询" :prefix-icon="Search"></el-input>
    <el-input style="width: 260px;margin-right: 10px" v-model="data.no" placeholder="请输入课程号查询" :prefix-icon="Search"></el-input>
    <el-input style="width: 260px" v-model="data.teacher" placeholder="请输入老师名称称查询" :prefix-icon="Search"></el-input>
    <el-button type="primary" style="margin-left: 10px" @click="load">查询</el-button>
    <el-button type="info" @click="reset">重置</el-button>
  </div>
  <div class="card" style="margin-bottom: 10px">
    <div style="margin-bottom: 10px">
      <el-button type="primary" @click="handleAdd">新增</el-button>
    </div>

    <div>
      <el-table :data="data.tableData" style="width: 100%">
        <el-table-column prop="id" label="序号" width="70" />
        <el-table-column prop="name" label="课程名称" />
        <el-table-column prop="no" label="课程编号"/>
        <el-table-column prop="descr" label="课程描述"/>
        <el-table-column prop="times" label="课时"/>
        <el-table-column prop="teacher" label="任课老师"/>

        <el-table-column label="操作">
          <template #default="scope">
            <el-button type="primary" plain @click="handleEdit(scope.row)">编辑</el-button>
            <el-button type="danger" plain @click="del(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>

    </div>
  </div>
  <div class="card">
    <el-pagination v-model:current-page="data.pageNum" v-model:page-size="data.pageSize"
                   @current-change="handelCurrentChange"
                   background layout="prev, pager, next" :total="data.total" />
  </div>

  <el-dialog v-model="data.formVisible" title="课程信息" width="35%">
    <el-form :data="data.form" label-width="100px" label-position="right" style="margin-right: 30px">
      <el-form-item label="课程名称">
        <el-input v-model="data.form.name" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="课程编号">
        <el-input v-model="data.form.no" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="课程描述">
        <el-input v-model="data.form.descr" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="课时">
        <el-input v-model="data.form.times" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="科任老师">
        <el-input v-model="data.form.teacher" autocomplete="off"></el-input>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="data.formVisible = false">取消</el-button>
        <el-button type="primary" @click="save">
          保存
        </el-button>
      </span>
    </template>
  </el-dialog>

</div>
</template>

<script setup>
import {reactive} from "vue";
import { Search } from '@element-plus/icons-vue'
import request from "../../utils/request";
import {ElMessage,ElMessageBox} from "element-plus";
const data = reactive({
  name:'',
  no: '',
  teacher:'',
  tableData:[],
  total:0,
  pageNum:1, //当前页码
  pageSize:5, // 每页数据个数
  formVisible:false,
  form:{}
})
const  load  = () => {
  request.get("/course/selectPage",{
    params:{
      pageNum:data.pageNum,
      pageSize:data.pageSize,
      name:data.name,
      no:data.no,
      teacher:data.teacher
    }
  }).then(res => {
    //有值就取值，如果没有或者空值就不取list里面的值
    data.tableData = res.data?.list || []
    //有值就取值，如果没有或者空值就不取total里面的值
    data.total = res.data?.total || 0
  })
}
//调用方法，调用后台数据
load();

const handelCurrentChange = (res) => {
  console.log(res);
  //调用方法，调用后台数据
  load();
}

const reset = () => {
  data.name = '',
  data.no = '',
  data.teacher = '',
  load()
}

const handleAdd = () =>{
  // 清空
  data.form = {}
  // 打卡弹窗
  data.formVisible = true
}

// 保存数据到后台
const save = () => {
  request.request({
    url: data.form.id ? '/course/update' : '/course/add',
    method: data.form.id ? 'PUT' : 'POST',
    data: data.form
  }).then(res => {
    if (res.code === '200') {
      load()    // 重新获取数据
      data.formVisible = false  // 关闭弹窗
      ElMessage.success("操作成功")
    } else {
      ElMessage.error(res.msg)
    }
  })
}
const handleEdit = (row) =>{
  data.form = JSON.parse(JSON.stringify(row))
  data.formVisible = true
}

const del = (id) => {
  ElMessageBox.confirm('删除数据后无法恢复，您确认删除吗？', '删除确认', { type: 'warning' }).then(res => {
    request.delete('/course/delete/' + id).then(res => {
      if (res.code === '200') {
        load()    // 重新获取数据
        ElMessage.success("操作成功")
      } else {
        ElMessage.error(res.msg)
      }
    })
  }).catch(res => {})
}
</script>

