<template>
  <div>
    <div class="box" style="margin-bottom: 5px">
      <el-input style="width: 240px; margin-right: 10px" v-model="data.name" placeholder="请输入名称查询"></el-input>
      <el-select v-model="data.categoryId" style="width: 240px; margin-right: 10px"  placeholder="请选择分类">
        <el-option v-for="item in data.categoryList" :key="item.id" :label="item.name" :value="item.id"></el-option>
      </el-select>
      <el-button type="primary" plain @click="load">查询</el-button>
      <el-button type="info" plain @click="reset">重置</el-button>
    </div>

    <div class="box" style="margin-bottom: 5px">
      <el-button type="primary" plain @click="handleAdd">新增</el-button>
      <el-button type="danger" plain @click="delBatch">批量删除</el-button>
    </div>

    <div class="box" style="margin-bottom: 5px">
      <el-table :data="data.tableData" strip @selection-change="handleSelectionChange">
        <!--选择框-->
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column prop="name" label="商品名称"></el-table-column>
        <el-table-column prop="categoryName" label="商品分类"></el-table-column>
        <el-table-column prop="img" label="商品图片">
          <template #default="scope">
            <el-image v-if="scope.row.img" :src="scope.row.img" :preview-src-list="[scope.row.img]" preview-teleported
                      style="width: 100px; height: 60px"></el-image>
          </template>
        </el-table-column>
        <el-table-column prop="originPrice" label="原价"></el-table-column>
        <el-table-column prop="store" label="库存"></el-table-column>
        <el-table-column prop="hasGroup" label="是否团购">
          <template #default="scope">
            <strong v-if="scope.row.hasGroup" style="color: red">是</strong>
            <span v-else>否</span>
          </template>
        </el-table-column>
        <el-table-column prop="groupPrice" label="团购价"></el-table-column>
        <el-table-column prop="hasFlash" label="是否秒杀">
          <template #default="scope">
            <strong v-if="scope.row.hasFlash" style="color: red">是</strong>
            <span v-else>否</span>
          </template>
        </el-table-column>
        <el-table-column prop="flashPrice" label="秒杀价"></el-table-column>
        <el-table-column prop="flashTime" label="秒杀截止时间" show-overflow-tooltip></el-table-column>
        <el-table-column prop="flashNum" label="秒杀名额"></el-table-column>
        <el-table-column prop="content" label="商品详情" width="100">
          <template #default="scope">
            <el-button plain type="primary" @click="preview(scope.row.content)">查看详情</el-button>
          </template>
        </el-table-column>
        <el-table-column prop="date" label="上架日期" show-overflow-tooltip></el-table-column>
        <el-table-column label="操作" header-align="center" width="160">
          <template #default="scope">
            <el-button type="primary" plain @click="handleEdit(scope.row)">编辑</el-button>
            <el-button type="danger" plain @click="del(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>


    <!-- 分页 -->
    <div class="box" v-if="data.total">
      <el-pagination @current-change="load" background layout="total, prev, pager, next"
                     :total="data.total" v-model:current-page="data.pageNum" v-model:page-size="data.pageSize" />
    </div>

    <!-- 编辑新增框 -->
    <el-dialog v-model="data.formVisible" title="商品信息" width="50%" destroy-on-close>
      <el-form :model="data.form" ref="formRef" :rules="data.rules" label-width="100px" style="padding: 20px">
        <el-form-item label="商品名称" prop="name">
          <el-input v-model="data.form.name" placeholder="商品名称"></el-input>
        </el-form-item>
        <el-form-item label="商品分类" prop="categoryId">
          <el-select v-model="data.form.categoryId" style="width: 100%">
            <el-option v-for="item in data.categoryList" :key="item.id" :label="item.name" :value="item.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="商品图片" prop="img">
          <el-upload :action="fileUploadUrl" :headers="{ myshoptoken: data.user.token }" :on-success="handleImgSuccess" >
            <el-button type="primary">上传商品图片</el-button>
          </el-upload>
        </el-form-item>
        <el-form-item label="原价" prop="originPrice">
          <el-input-number :min="0" v-model="data.form.originPrice" placeholder="原价"></el-input-number>
        </el-form-item>
        <el-form-item label="库存" prop="store">
          <el-input v-model="data.form.store" placeholder="库存"></el-input>
        </el-form-item>
        <el-form-item label="是否团购" prop="hasGroup">
          <el-switch v-model="data.form.hasGroup"></el-switch>
        </el-form-item>
        <el-form-item label="团购价" prop="groupPrice" v-if="data.form.hasGroup">
          <el-input-number :min="0" v-model="data.form.groupPrice" placeholder="团购价"></el-input-number>
        </el-form-item>
        <el-form-item label="是否秒杀" prop="hasFlash">
          <el-switch v-model="data.form.hasFlash"></el-switch>
        </el-form-item>
        <div v-if="data.form.hasFlash">
          <el-form-item label="秒杀价" prop="flashPrice" >
            <el-input-number :min="0" v-model="data.form.flashPrice" placeholder="秒杀价"></el-input-number>
          </el-form-item>
          <el-form-item label="秒杀名额" prop="flashNum">
            <el-input-number :min="0" v-model="data.form.flashNum" placeholder="秒杀名额"></el-input-number>
          </el-form-item>
          <el-form-item label="秒杀截止时间" prop="flashTime">
            <el-date-picker format="YYYY-MM-DD HH:mm:ss" value-format="YYYY-MM-DD HH:mm:ss" v-model="data.form.flashTime" type="datetime" placeholder="秒杀截止时间" />
          </el-form-item>
        </div>
        <el-form-item label="商品详情" prop="content">
          <div style="border: 1px solid #ccc; width: 100%">
            <Toolbar
                style="border-bottom: 1px solid #ccc"
                :editor="editorRef"
                :mode="mode"
            />
            <Editor
                style="height: 500px; overflow-y: hidden;"
                v-model="data.form.content"
                :mode="mode"
                :defaultConfig="editorConfig"
                @onCreated="handleCreated"
            />
          </div>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="data.formVisible = false">取消</el-button>
        <el-button type="primary" @click="save">保存</el-button>
      </template>
    </el-dialog>

    <el-dialog v-model="data.formVisible1" title="商品详情" width="50%" destroy-on-close>
      <div style="padding: 10px" v-html="data.content"></div>
      <template #footer>
        <el-button type="primary" @click="data.formVisible1 = false">关闭</el-button>
      </template>
    </el-dialog>

  </div>
</template>

<script setup>
import {onBeforeUnmount, reactive, ref, shallowRef} from "vue";
import request from "@/utils/request.js";
import {ElMessage, ElMessageBox} from "element-plus";

import '@wangeditor/editor/dist/css/style.css' // 引入 css
import { Editor, Toolbar } from '@wangeditor/editor-for-vue'

const data = reactive({
  user: JSON.parse(localStorage.getItem('myshop-user') || '{}'),
  name: null,
  categoryId: null,
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
    categoryId: [
      { required: true, message: '请选择分类', trigger: 'blur' }
    ],
    originPrice: [
      { required: true, message: '请输入原价', trigger: 'blur' }
    ],
    store: [
      { required: true, message: '请输入库存', trigger: 'blur' }
    ]
  },
  ids: [],
  categoryList: [],
  content: '',
  formVisible1: false
})
// 文件上传地址
const fileUploadUrl = import.meta.env.VITE_BASE_URL + '/files/upload'

const formRef = ref()

/* wangEditor5 初始化开始 */
const editorRef = shallowRef()  // 编辑器实例，必须用 shallowRef
const mode = 'default'
const editorConfig = { MENU_CONF: {} }

// 图片上传配置
editorConfig.MENU_CONF['uploadImage'] = {
  headers: {
    myshoptoken: data.user.token,
  },
  server: import.meta.env.VITE_BASE_URL + '/files/wang/upload',  // 服务端图片上传接口
  fieldName: 'file'  // 服务端图片上传接口参数
}
// 组件销毁时，也及时销毁编辑器，否则可能会造成内存泄漏
onBeforeUnmount(() => {
  const editor = editorRef.value
  if (editor == null) return
  editor.destroy()
})
// 记录 editor 实例，重要！
const handleCreated = (editor) => {
  editorRef.value = editor
}
/* wangEditor5 初始化结束 */

const load = () => {
  request.get('/goods/selectPage', {
    params: {
      pageNum: data.pageNum,
      pageSize: data.pageSize,
      name: data.name,
      categoryId: data.categoryId
    }
  }).then(res => {
    data.tableData = res.data.list
    data.total = res.data.total
  })
}

const loadCategory = () => {
  request.get('/category/selectAll').then(res => {
    data.categoryList = res.data
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
  request.post('/goods/add', data.form).then(res => {
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
  request.put('/goods/update', data.form).then(res => {
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
    request.delete('/goods/delete/' + id).then(res => {
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
  data.categoryId = null
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
    request.delete('/goods/deleteBatch', { data: data.ids }).then(res => {
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

const preview = (content) => {
  data.content = content
  data.formVisible1 = true
}

load()
loadCategory()
</script>