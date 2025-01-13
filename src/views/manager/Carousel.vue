<template>
  <div>
    <div class="box" style="margin-bottom: 5px">
      <el-input style="width: 240px; margin-right: 10px" v-model="data.goodsName" placeholder="请输入商品名称查询"></el-input>
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
        <el-table-column prop="goodsName" label="商品名称" />
        <el-table-column prop="img" label="推荐图片">
          <template #default="scope">
            <el-image v-if="scope.row.img" :src="scope.row.img" :preview-src-list="[scope.row.img]" preview-teleported
                      style="width: 100px; height: 60px"></el-image>
          </template>
        </el-table-column>
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

    <el-dialog v-model="data.formVisible" title="广告信息" width="40%" destroy-on-close>
      <el-form :model="data.form" ref="formRef" :rules="data.rules" label-width="90px" style="padding: 20px">
        <el-form-item label="推荐商品" prop="goodsId">
          <el-select v-model="data.form.goodsId" style="width: 100%">
            <el-option v-for="item in data.goodsList" :key="item.id" :label="item.name" :value="item.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="推荐图片" prop="img">
          <el-upload :action="fileUploadUrl" :headers="{ myshoptoken: data.user.token }" :on-success="handleImgSuccess" >
            <el-button type="primary">上传商品图片</el-button>
          </el-upload>
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
    user: JSON.parse(localStorage.getItem('myshop-user') || '{}'),
    goodsName: null,
    pageNum: 1,
    pageSize: 10,
    total: 0,
    tableData: [],
    formVisible: false,
    form: {},
    rules: {
      goodsId: [
        { required: true, message: '请选择商品', trigger: 'change' }
      ],
      img: [
        { required: true, message: '请上传图片', trigger: 'blur' }
      ],
    },
    ids: [],
    goodsList: []
  })
  const fileUploadUrl = import.meta.env.VITE_BASE_URL + '/files/upload'

  const formRef = ref()

  const load = () => {
    request.get('/carousel/selectPage', {
      params: {
        pageNum: data.pageNum,
        pageSize: data.pageSize,
        goodsName: data.goodsName
      }
    }).then(res => {
      data.tableData = res.data.list
      data.total = res.data.total
    })
  }

  const loadGoods = () => {
    request.get('/goods/selectAll').then(res => {
      data.goodsList = res.data
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
    request.post('/carousel/add', data.form).then(res => {
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
    request.put('/carousel/update', data.form).then(res => {
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
      request.delete('/carousel/delete/' + id).then(res => {
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
    data.goodsName = null
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
      request.delete('/carousel/deleteBatch', { data: data.ids }).then(res => {
        if (res.code === '200') {  // 表示请求成功
          ElMessage.success('删除成功')
          load()
        } else {
          ElMessage.error(res.msg)
        }
      })
    }).catch(err => {})
  }

  const handleImgSuccess = (res) => {
    data.form.img = res.data
  }

  load()
  loadGoods()
</script>