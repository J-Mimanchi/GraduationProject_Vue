<template>
  <div>
    <div class="box" style="margin-bottom: 5px">
      <el-table :data="data.tableData" stripe>
        <el-table-column prop="goodsName" label="商品名称" width="200px" fixed="left">
          <template v-slot="scope">
            <a :href="'/front/goodsDetail?id=' + scope.row.goodsId" target="_blank">{{ scope.row.goodsName }}</a>
          </template>
        </el-table-column>
        <el-table-column prop="content" label="评价内容" />
        <el-table-column prop="score" label="评分" />
        <el-table-column prop="createTime" label="评价时间" />
        <el-table-column prop="reply" label="回复内容" show-overflow-tooltip/>
        <el-table-column label="操作" header-align="center" width="180" fixed="right">
          <template #default="scope">
            <el-button plain type="primary" @click="handleEdit(scope.row)">回复</el-button>
            <el-button plain type="danger" @click="del(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div class="box" v-if="data.total">
      <el-pagination @current-change="load" background layout="total, prev, pager, next"
                     :total="data.total" v-model:current-page="data.pageNum" v-model:page-size="data.pageSize" />
    </div>
    <el-dialog title="回复信息" v-model="data.formVisible" width="40%" :close-on-click-modal="false" destroy-on-close>
      <el-form :model="data.form" label-width="80px"  style="padding: 20px 30px" ref="formRef">
        <el-form-item label="回复内容" prop="content">
          <el-input type="textarea" :rows="4" v-model="data.form.reply" placeholder="回复内容"></el-input>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="data.formVisible = false">取消</el-button>
          <el-button type="primary" @click="reply">提交</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import {reactive, onMounted} from "vue";
import request from "@/utils/request.js";
import {ElMessage, ElMessageBox} from "element-plus";
import router from "@/router/index.js";

const data = reactive({
  user: JSON.parse(localStorage.getItem('myshop-user') || '{}'),
  pageNum: 1,
  pageSize: 10,
  total: 0,
  tableData: [],
  form: {},
  formVisible: false
})

const load = () => {
  request.get('/comment/selectPage', {
    params: {
      pageNum: data.pageNum,
      pageSize: data.pageSize,
    }
  }).then(res => {
    data.tableData = res.data?.list
    data.total = res.data.total
  })
}
const del = (id) => {
  ElMessageBox.confirm('删除后数据无法恢复，您确定删除吗？', '删除确认', { type: 'warning' }).then(res => {
    request.delete('/orders/delete/' + id).then(res => {
      if (res.code === '200') {  // 表示请求成功
        ElMessage.success('删除成功')
        load()
      } else {
        ElMessage.error(res.msg)
      }
    })
  }).catch(err => {})
}
const handleEdit = (row) => {
  data.form = JSON.parse(JSON.stringify(row))
  data.formVisible = true
}

const reply = () => {
  request.put('/comment/update', data.form).then(res => {
    if (res.code === '200') {
      ElMessage.success('回复成功')
      data.formVisible = false
      load()
    } else {
      ElMessage.error(res.msg)
    }
  })
}
load()



</script>

<style scope>
</style>