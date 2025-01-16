<template>
  <div>
    <div class="box" style="margin-bottom: 5px">
      <el-input style="width: 240px; margin-right: 10px" v-model="data.orderNo" placeholder="请输入订单编号查询"></el-input>
      <el-button type="primary" plain @click="load">查询</el-button>
      <el-button type="info" plain @click="reset">重置</el-button>
    </div>

    <div class="box" style="margin-bottom: 5px">
      <el-button type="danger" plain @click="delBatch">批量删除</el-button>
    </div>

    <div class="box" style="margin-bottom: 5px">
      <el-table :data="data.tableData" stripe @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="goodsName" label="商品名称" width="200px"/>
        <el-table-column prop="goodsImg" label="商品图片">
          <template #default="scope">
            <el-image v-if="scope.row.goodsImg" :src="scope.row.goodsImg" :preview-src-list="[scope.row.goodsImg]" preview-teleported
                      style="width: 100px; height: 60px"></el-image>
          </template>
        </el-table-column>
        <el-table-column prop="goodsPrice" label="商品单价" />
        <el-table-column prop="num" label="购买数量" />
        <el-table-column prop="total" label="商品总价" />
        <el-table-column prop="orderNo" label="订单编号" />
        <el-table-column prop="payNo" label="支付单号" />
        <el-table-column prop="payTime" label="支付时间" />
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
            <span v-if="scope.row.type === 'GROUP'">团购订单</span>
            <span v-if="scope.row.type === 'FLASH'">秒杀订单</span>
          </template>
        </el-table-column>
        <el-table-column prop="userName" label="下单人" />
        <el-table-column prop="createTime" label="创建时间" show-overflow-tooltip />
        <el-table-column label="操作" header-align="center" width="180" fixed="right">
          <template #default="scope">
            <el-button plain type="primary" @click="update(scope.row)" v-if="scope.row.status === 'NOT_SEND'">发货</el-button>
            <el-button plain type="danger" @click="del(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div class="box" v-if="data.total">
      <el-pagination @current-change="load" background layout="total, prev, pager, next"
                     :total="data.total" v-model:current-page="data.pageNum" v-model:page-size="data.pageSize" />
    </div>

  </div>
</template>

<script setup>
  import {reactive, ref} from "vue";
  import request from "@/utils/request.js";
  import {ElMessage, ElMessageBox} from "element-plus";

  const data = reactive({
    user: JSON.parse(localStorage.getItem('myshop-user') || '{}'),
    orderNo: null,
    pageNum: 1,
    pageSize: 10,
    total: 0,
    tableData: [],
    ids: [],
  })

  const load = () => {
    request.get('/orders/selectPage', {
      params: {
        pageNum: data.pageNum,
        pageSize: data.pageSize,
        orderNo: data.orderNo
      }
    }).then(res => {
      data.tableData = res.data.list
      data.total = res.data.total
    })
  }

  // 发货
  const update = (row) => {
    row.status = 'NOT_ACCEPT'  // 修改状态为待收货
    request.put('/orders/update', row).then(res => {
      if (res.code === '200') {  // 表示请求成功
        ElMessage.success('发货成功')
        //刷新表格数据
        load()
      } else {  // 表示错误
        ElMessage.error(res.msg)
      }
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

  const reset = () => {
    data.orderNo = null
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
      request.delete('/orders/deleteBatch', { data: data.ids }).then(res => {
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