<template>
  <div style="width: 70%; margin: 20px auto">
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
            <el-button plain type="danger" @click="del(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div v-if="data.total">
      <el-pagination @current-change="load" background layout="total, prev, pager, next"
                     :total="data.total" v-model:current-page="data.pageNum" v-model:page-size="data.pageSize" />
    </div>
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
})

const load = () => {
  request.get('/comment/selectPage', {
    params: {
      pageNum: data.pageNum,
      pageSize: data.pageSize,
      userId: data.user.id
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
load()



</script>

<style scope>
</style>