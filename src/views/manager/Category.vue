<template>
  <div>
    <div class="box" style="margin-bottom: 5px">
      <el-input style="width: 240px; margin-right: 10px" v-model="data.name" placeholder="请输入名称查询"></el-input>
      <el-button type="primary" plain @click="load">查询</el-button>
      <el-button type="info" plain @click="reset">重置</el-button>
    </div>

    <div class="box" style="margin-bottom: 5px">
      <el-button type="primary" plain @click="handleAdd">新增</el-button>
      <el-button type="danger" plain @click="delBatch">批量删除</el-button>
    </div>

    <div class="box" style="margin-bottom: 5px">
      <el-table :data="data.tableData" stripe @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="name" label="名称" />
        <el-table-column label="操作" header-align="center" width="180">
          <template #default="scope">
            <el-button plain type="primary" @click="handleEdit(scope.row)">编辑</el-button>
            <el-button plain type="danger" @click="del(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div class="box" v-if="data.total">
      <el-pagination @current-change="load" background layout="total, prev, pager, next"
                     :total="data.total" v-model:current-page="data.pageNum" v-model:page-size="data.pageSize" />
    </div>

    <el-dialog v-model="data.formVisible" title="分类信息" width="40%" destroy-on-close>
      <el-form :model="data.form" ref="formRef" :rules="data.rules" label-width="70px" style="padding: 20px">
        <el-form-item label="名称" prop="name">
          <el-input v-model="data.form.name" autocomplete="off" placeholder="请输入名称" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="data.formVisible = false">取消</el-button>
        <el-button type="primary" @click="save">保存</el-button>
      </template>
    </el-dialog>

  </div>
</template>

<script setup>
import {reactive, ref} from "vue";
import request from "@/utils/request.js";
import {ElMessage, ElMessageBox} from "element-plus";

const data = reactive({
  category: JSON.parse(localStorage.getItem('ttl-category') || '{}'),
  name: null,
  pageNum: 1,
  pageSize: 10,
  total: 0,
  tableData: [],
  formVisible: false,
  form: {},
  rules: {
    name: [
      { required: true, message: '请输入名称', trigger: 'blur' }
    ],
  },
  ids: []
})

const formRef = ref()
const formRef1 = ref()

const load = () => {
  request.get('/category/selectPage', {
    params: {
      pageNum: data.pageNum,
      pageSize: data.pageSize,
      name: data.name
    }
  }).then(res => {
    data.tableData = res.data.list
    data.total = res.data.total
  })
}

const handleAdd = () => {
  data.form = {}
  data.formVisible = true
}

const handleEdit = (row) => {
  // 先对行数据进行深度拷贝
  data.form = JSON.parse(JSON.stringify(row))
  data.formVisible = true
}

const add = () => {
  request.post('/category/add', data.form).then(res => {
    if (res.code === '200') {  // 表示请求成功
      ElMessage.success('保存成功')
      //刷新表格数据
      load()
      data.formVisible = false
    } else {  // 表示错误
      ElMessage.error(res.msg)
    }
  })
}

const update = () => {
  request.put('/category/update', data.form).then(res => {
    if (res.code === '200') {  // 表示请求成功
      ElMessage.success('保存成功')
      //刷新表格数据
      load()
      data.formVisible = false
    } else {  // 表示错误
      ElMessage.error(res.msg)
    }
  })
}

const save = () => {
  formRef.value.validate((valid) => {
    if (valid) {
      if (data.form.id) {  // 有id表示数据存在，那么就是更新
        update()
      } else {  // 否则就是新增
        add()
      }
    }
  })
}

const del = (id) => {
  ElMessageBox.confirm('删除后数据无法恢复，您确定删除吗？', '删除确认', { type: 'warning' }).then(res => {
    request.delete('/category/delete/' + id).then(res => {
      if (res.code === '200') {  // 表示请求成功
        ElMessage.success('删除成功')
        load()
      } else {
        ElMessage.error(res.msg)
      }
    })
  }).catch(err => {})
}

const reset = () => {
  data.name = null
  load()
}

const handleSelectionChange = (value) => {
  data.ids = value.map(v => v.id)
}

const delBatch = () => {
  if (data.ids.length === 0) {
    ElMessage.warning('请选择数据')
    return
  }
  ElMessageBox.confirm('删除后数据无法恢复，您确定删除吗？', '删除确认', { type: 'warning' }).then(res => {
    request.delete('/category/deleteBatch', { data: data.ids }).then(res => {
      if (res.code === '200') {  // 表示请求成功
        ElMessage.success('删除成功')
        load()
      } else {
        ElMessage.error(res.msg)
      }
    })
  }).catch(err => {})
}

load()
</script>