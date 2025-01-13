<template>
    <div>
        <div class="box" style="margin-bottom: 5px;">
            <el-input style="width: 240px; margin-right: 10px;" v-model="data.username" placeholder="请输入账号查询"></el-input>
            <el-button type="primary" plain @click="load">查询</el-button>
            <el-button type="info" plain @click="reset">重置</el-button>
        </div>

        <div class="box" style="margin-bottom: 5px;">
            <el-button type="primary" plain @click="handleAdd">新增</el-button>
            <el-button type="danger" plain @click="delBatch">批量删除</el-button>
        </div>

        <div class="box" style="margin-bottom: 5px;">
            <el-table :data="data.tableData" stripe @selection-change="handleSelectionChange">
                <el-table-column type="selection" width="55"/>
                <el-table-column prop="username" label="账户"/>
                <el-table-column prop="name" label="名称"/>
                <el-table-column prop="role" label="角色"/>
                <el-table-column prop="avatar" label="头像">
                    <template #default="scope">
                        <el-image v-if="scope.row.avatar" :src="scope.row.avatar" :preview-src-list="[scope.row.avatar]" preview-teleported style="width: 50px; height: 50px">

                        </el-image>
                    </template>
                </el-table-column>
                <el-table-column prop="phone" label="手机号"/>
                <el-table-column prop="email" label="邮箱"/>
                <el-table-column label="操作" header-align="center" width="280">
                <template #default="scope">
                    <el-button plain type="primary" @click="handlePassword(scope.row)">修改密码</el-button>
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

        <el-dialog v-model="data.formVisible" title="用户信息" width="40%">
            <el-form :model="data.form" ref="formRef":rules="data.rules" label-width="80px" style="padding: 20px">
                <el-form-item label="账号" prop="username">
                    <el-input v-model="data.form.username" autocomplete="off" placeholder="请输入账号"/>
                </el-form-item>
                <el-form-item label="名称">
                    <el-input v-model="data.form.name" autocomplete="off" placeholder="请输入名称"/>
                </el-form-item>
                <el-form-item label="角色" prop="role">
                    <el-select v-model="data.form.role" placeholder="请选择角色" style="width: 100%;">
                        <el-option label="管理员" value="ADMIN"/>
                        <el-option label="用户" value="USER"/>
                    </el-select>
                </el-form-item>
                <el-form-item label="头像">
                    <el-upload :action="fileUploadUrl" :headers="{ myshoptoken: data.user.token }" :on-success="handleAvatarSuccess">
                        <el-button size="small" type="primary">点击上传</el-button>
                    </el-upload>
                </el-form-item>
                    <el-input v-model="data.form.avatar" autocomplete="off" placeholder="请输入头像"/>
                <el-form-item label="手机号" prop="phone">
                    <el-input v-model="data.form.phone" autocomplete="off" placeholder="请输入手机号" />
                </el-form-item>
                <el-form-item label="邮箱">
                    <el-input v-model="data.form.email" autocomplete="off" placeholder="请输入邮箱" />
                 </el-form-item>
            </el-form>
            <template #footer>
                <el-button @click="data.formVisible = false">取消</el-button>
                <el-button type="primary" @click="save">保存</el-button>
            </template>
        </el-dialog>
        <el-dialog v-model="data.formVisible1" title="修改密码" width="40%">
            <el-form :model="data.form" ref="formRef1":rules="data.rules1" label-width="80px" style="padding: 20px">
                <el-form-item label="新密码" prop="newPassword">
                    <el-input show-password v-model="data.form.newPassword" autocomplete="off" placeholder="请输入密码"/>
                </el-form-item>
                <el-form-item label="确认密码" prop="confirmPassword">
                    <el-input show-password v-model="data.form.confirmPassword" autocomplete="off" placeholder="请确认密码"/>
                </el-form-item>
            </el-form>
            <template #footer>
                <el-button @click="data.formVisible1 = false">取消</el-button>
                <el-button type="primary" @click="savePassword">保存</el-button>
            </template>
        </el-dialog>
    </div>
</template>

<script setup>

import {reactive, ref} from "vue";
import request from '@/utils/request.js';
import {ElMessage, ElMessageBox} from "element-plus";


const fileUploadUrl = import.meta.env.VITE_BASE_URL + '/files/upload';

const validatePass = (rule, value, callback) => {
  if (!value) {
    callback(new Error('请确认密码'));
  } else if (value !== data.form.newPassword) {
    callback(new Error("确认密码与原密码不匹配!"));
  } else {
    callback();
  }
}

const data = reactive({
    user:JSON.parse(localStorage.getItem('myshop-user')||'{}'),
    username: null,
    pageNum: 1,
    pageSize: 10,
    total: 0,
    tableData:[],
    formVisible: false,
    formVisible1: false,
    form: {},
    ids: [],

    rules: {
        username: [
            {required: true, message: '请输入账号', trigger: 'blur'}
        ],
        role: [
            {required: true, message: '请选择角色', trigger: 'change'}
        ],

    },
    rules1: {
        newPassword: [
            {required: true, message: '请输入新密码', trigger: 'blur'}
        ],
        confirmPassword: [
            {required: true,validator:validatePass, trigger: 'blur'}
        ],
    },
  });

const formRef = ref();
const formRef1 = ref();

const load = () => {
    request.get('/user/selectPage',{
        params: {
            pageNum: data.pageNum,
            pageSize: data.pageSize,
            username: data.username
        }
    }).then(res => {
        data.tableData = res.data.list;
        data.total = res.data.total;
    });
}  

const handlePageChange = (page) => {
    data.pageNum = page;
    load();
}

const reset = () => {
    data.username = null;
    load();
}

const handleAdd = () => {
    data.form = {};
    data.formVisible = true;
}

const add = () => {
    formRef.value.validate((valid) => {
        if(valid) {
            request.post('/user/add',data.form).then(res => {
                if(res.code == 200){
                    ElMessage.success('保存成功');
                    data.formVisible = false;
                    load();
                }else{
                    ElMessage.error(res.msg);
                }
            })
        }
    });
}

const update = () => {
    formRef.value.validate((valid) => {
        if(valid) {
            request.put('/user/update',data.form).then(res => {
                if(res.code == 200){
                    ElMessage.success('保存成功');
                    data.formVisible = false;
                    load();
                }else{
                    ElMessage.error(res.msg);
                }
            })
        }
    });
}

const save = () => {
    formRef.value.validate((valid) => {
        if(valid) {
            if(data.form.id){//有id表示数据已存在，进行修改，否则进行新增
                update();
            }else{
                add();
            }
        }
    });
}

const handlePassword = (row) => {
    data.form = JSON.parse(JSON.stringify(row));
    data.formVisible1 = true;
}

const handleEdit = (row) => {
    // 深拷贝
    data.form = JSON.parse(JSON.stringify(row));
    data.formVisible = true;
}

const del = (id) => {
    ElMessageBox.confirm('删除后数据无法恢复，您确定删除吗？', '删除确认', { type: 'warning' }).then(() => {
        request.delete('/user/delete/'+id).then(res => {
            if(res.code == 200){
                ElMessage.success('删除成功');
                load();
            }else{
                ElMessage.error(res.msg);
            }
        })
    }).catch(() => {
        ElMessage.info('取消删除');
    })
}

const savePassword = () => {
    formRef1.value.validate((valid) => {
        if(valid) {
            request.put('/updatePassword',data.form).then(res =>{
                if(res.code==200){
                    ElMessage.success('修改成功');
                    data.formVisible1 = false;
                }else{
                    ElMessage.error(res.msg);
                }
            })
        }
    });
}

const handleSelectionChange = (value) => {
    data.ids = value.map(item => item.id);
}

const delBatch = () => {
    if (data.ids.length === 0) {
      ElMessage.warning('请选择数据')
      return
    }
    ElMessageBox.confirm('删除后数据无法恢复，您确定删除吗？', '删除确认', { type: 'warning' }).then(res => {
      request.delete('/user/deleteBatch', { data: data.ids }).then(res => {
        if (res.code === '200') {  // 表示请求成功
          ElMessage.success('删除成功')
          load()
        } else {
          ElMessage.error(res.msg)
        }
      })
    }).catch(err => {
        ElMessage.info('取消删除');
    })
  }

const handleAvatarSuccess = (res) => {
    if(res.code == 200){
        data.form.avatar = res.data;
    }else{
        ElMessage.error(res.msg);
    }
}

load();
</script>