<template>
  <div style="width: 80%; margin: 20px auto">
    <div class="box" style="margin-bottom: 5px">
      <el-table :data="data.tableData" stripe>
        <el-table-column prop="goodsName" label="商品名称" width="200px" fixed="left">
          <template v-slot="scope">
            <a :href="'/front/goodsDetail?id=' + scope.row.goodsId" target="_blank">{{ scope.row.goodsName }}</a>
          </template>
        </el-table-column>
        <el-table-column prop="goodsImg" label="商品图片" width="100px">
          <template #default="scope">
            <el-image v-if="scope.row.goodsImg" :src="scope.row.goodsImg" :preview-src-list="[scope.row.goodsImg]" preview-teleported
                      style="width: 60px; height: 60px"></el-image>
          </template>
        </el-table-column>
        <el-table-column prop="goodsPrice" label="商品单价" />
        <el-table-column prop="num" label="购买数量" />
        <el-table-column prop="total" label="商品总价" />
        <el-table-column prop="orderNo" label="订单编号" show-overflow-tooltip/>
        <el-table-column prop="payNo" label="支付单号" show-overflow-tooltip/>
        <el-table-column prop="payTime" label="支付时间" show-overflow-tooltip/>
        <el-table-column prop="status" label="订单状态">
          <template #default="scope">
            <el-tag type="danger" v-if="scope.row.status === 'CANCEL'">已取消</el-tag>
            <el-tag type="primary" v-if="scope.row.status === 'NOT_PAY'">待支付</el-tag>
            <el-tag type="primary" v-if="scope.row.status === 'IN_GROUP'">拼团中</el-tag>
            <el-tag type="primary" v-if="scope.row.status === 'NOT_SEND'">待发货</el-tag>
            <el-tag type="primary" v-if="scope.row.status === 'NOT_ACCEPT'">待收货</el-tag>
            <el-tag type="success" v-if="scope.row.status === 'DONE'">已完成</el-tag>
            <el-tag type="danger" v-if="scope.row.status === 'REFUND_DONE'">已退款</el-tag>
            <el-tag type="success" v-if="scope.row.status === 'COMMENT_DONE'">已评价</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="type" label="订单类型">
          <template #default="scope">
            <span v-if="scope.row.type === 'COMMON'">普通订单</span>
            <span v-if="scope.row.type === 'GROUP'">普通订单</span>
            <span v-if="scope.row.type === 'FLASH'">秒杀订单</span>
          </template>
        </el-table-column>
        <el-table-column prop="userName" label="下单人" />
        <el-table-column prop="createTime" label="创建时间" show-overflow-tooltip />
        <el-table-column label="操作" header-align="center" width="240" fixed="right">
          <template #default="scope">
            <el-button plain type="primary" @click="update(scope.row, 'DONE')" v-if="scope.row.status === 'NOT_ACCEPT'">确认收货</el-button>
            <el-button plain type="primary" @click="initComment(scope.row)" v-if="scope.row.status === 'DONE'">评价</el-button>
            <el-button plain type="primary" @click="refund(scope.row.orderNo)" v-if="scope.row.status === 'DONE'">退款</el-button>
            <el-button plain type="danger" @click="del(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div v-if="data.total">
      <el-pagination @current-change="load" background layout="total, prev, pager, next"
                     :total="data.total" v-model:current-page="data.pageNum" v-model:page-size="data.pageSize" />
    </div>

    <el-dialog title="评价信息" v-model="data.formVisible" width="40%" :close-on-click-modal="false" destroy-on-close>
      <el-form :model="data.form" label-width="80px"  style="padding: 20px 30px" ref="formRef">
        <el-form-item label="评价内容" prop="content">
          <el-input type="textarea" :rows="4" v-model="data.form.content" placeholder="评价内容"></el-input>
        </el-form-item>
        <el-form-item label="评分" prop="score">
          <el-rate v-model="data.form.score" />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="data.formVisible = false">取消</el-button>
          <el-button type="primary" @click="comment">提交</el-button>
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
  formVisible: false,
  orderData: {}
})

const load = () => {
  request.get('/orders/selectPage', {
    params: {
      pageNum: data.pageNum,
      pageSize: data.pageSize,
      userId: data.user.id
    }
  }).then(res => {
    data.tableData = res.data.list
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
const update = (row, status) => {
  row.status = status  // 修改状态为已完成
  request.put('/orders/update', row).then(res => {
    if (res.code === '200') {  // 表示请求成功
      //刷新表格数据
      load()
    } else {  // 表示错误
      ElMessage.error(res.msg)
    }
  })
}
const refund = (orderNo) => {
  request.get('/alipay/refund?orderNo=' + orderNo).then(res => {
    if (res.code === '200') {
      ElMessage.success('退款成功')
      //刷新表格数据
      load()
    } else {
      ElMessage.error(res.msg)
    }
  })
}

const initComment = (orderData) => {
  data.orderData = orderData
  data.form = {}
  data.form.userId = data.user.id
  data.form.goodsId = orderData.goodsId
  data.formVisible = true
}

const comment = () => {
  request.post('/comment/add', data.form).then(res => {
    if (res.code === '200') {
      ElMessage.success('评价成功')
      data.formVisible = false
      // 更新一下订单的状态
      update(data.orderData, 'COMMENT_DONE')
    } else {
      ElMessage.error(res.msg)
    }
  })
}
load()



</script>

<style scope>
.overflow {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis
}
</style>