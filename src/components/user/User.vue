<template>
    <div class="">
        <Bread></Bread>
        <el-card class="box-card">
            <template #header>
                <div class="card-header">
                    <el-button class="white" color="#13b9c9" @click="add_">添加用户</el-button>
                </div>
            </template>
            <el-table :data="userList" border stripe style="width: 100%" show-overflow-tooltip :header-cell-style="{
                backgroundColor: 'rgba(19, 185, 201, 0.7)',
                color: 'rgb(70, 69, 69)'
            }">
                <el-table-column prop="name" label="姓名" width="" />
                <el-table-column prop="sex" label="性别" width="" />
                <el-table-column prop="phone" label="电话" width="200px" />
                <el-table-column prop="addr" label="地址" />
                <el-table-column label="分类操作" width="380">
                    <template #default="scope">
                        <el-button class="white" color="#13b9c9" @click="edit_(scope.row)">修改用户</el-button>
                        <el-button class="white" color="#13b9c9" @click="detail_(scope.row)">用户详情</el-button>
                        <el-popconfirm title="确定删除吗?" @confirm="delete_(scope.row.id)">
                            <template #reference>
                                <el-button class="white" color="#13b9c9">删除用户</el-button>
                            </template>
                        </el-popconfirm>

                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页器 -->
            <el-pagination v-model:current-page="pno" v-model:page-size="count" :page-sizes="[2, 5, 8, 15]"
                layout="total, sizes, prev, pager, next, jumper" :total="total" @size-change="handleSizeChange"
                @current-change="handleCurrentChange" />
        </el-card>
        <!-- dialog对话框 -->
        <el-dialog v-model="dialogVisible" :title="dialogTitle" width="60%" @close="dialogClose">
            <el-form ref="ruleFormRef" :model="ruleForm" :rules="rules" label-width="120px" status-icon>
                <el-form-item label="姓名" prop="name">
                    <el-input v-model="ruleForm.name" v-if="dialogTitle !== '用户详情'" />
                    <span v-else>{{ ruleForm.name }}</span>
                </el-form-item>
                <el-form-item label="手机号" prop="phone">
                    <el-input v-model="ruleForm.phone" v-if="dialogTitle !== '用户详情'" />
                    <span v-else>{{ ruleForm.phone }}</span>
                </el-form-item>
                <el-form-item label="性别" prop="sex">
                    <el-select v-model="ruleForm.sex" clearable placeholder="请选择" v-if="dialogTitle !== '用户详情'">
                        <el-option label="男" value="男" />
                        <el-option label="女" value="女" />

                    </el-select>
                    <span v-else>{{ ruleForm.sex }}</span>
                </el-form-item>
                <el-form-item label="照片" prop="id_pic">
                    <el-input v-model="ruleForm.id_pic" v-if="dialogTitle !== '用户详情'" />
                    <span v-else><img :src="ruleForm.id_pic" alt=""></span>
                </el-form-item>
                <el-form-item label="地址" prop="addr">
                    <el-input v-model="ruleForm.addr" type="textarea" v-if="dialogTitle !== '用户详情'" />
                    <span v-else v-html="ruleForm.addr"></span>
                </el-form-item>

            </el-form>
            <template #footer>
                <span class="dialog-footer" v-if="dialogTitle !== '用户详情'">
                    <el-button @click="dialogVisible = false">取消</el-button>
                    <el-button type="primary" @click="dialogEnter">
                        确定
                    </el-button>
                </span>
            </template>
        </el-dialog>
    </div>
</template>

<script lang="ts" setup>
import Bread from '../Bread.vue'
import { ElMessage } from 'element-plus';
import { ref, reactive, getCurrentInstance, onMounted } from 'vue'
import type { FormInstance, FormRules } from 'element-plus'
let { proxy }: any = getCurrentInstance()
let userList = ref([])
let pno = ref(1)
let count = ref(3)
let total = ref(10)
const getUserList = async () => {
    let { data: res } = await proxy.$http.get(`myuser/list?pno=${pno.value}&count=${count.value}`)
    // console.log(res);

    if (res.code !== 200) {
        ElMessage({
            message: '请求失败',
            type: 'error',
            duration: 500
        })
    } else {
        userList.value = res.data
        total.value = res.total
        pno.value = res.pno
    }
}
// 分页器
const handleSizeChange = (val: any) => {
    count.value = val
    getUserList()
}
const handleCurrentChange = (val: any) => {
    pno.value = val
    getUserList()
}
// dialog
let dialogVisible = ref(false)
const dialogTitle = ref('')
// from表单
let ruleFormRef = ref<FormInstance>()
let ruleForm = reactive({
    name: '',
    phone: '',

    sex: '',
    id_pic: '',
    addr: '',

})

// 验证手机号
var checkMobile = (rule: any, value: any, callback: any) => {
    const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|166|17[3678]|18[0-9]|14[57])[0-9]{8}$/
    if (regMobile.test(value)) {
        return callback()
    }
    return callback(new Error('请输入正确的手机号'))
}
let rules = reactive<FormRules>({
    name: [{ required: true, message: '请输入姓名', trigger: 'blur' }],
    phone: [{ required: true, message: '请填写手机号', trigger: 'blur' }, { validator: checkMobile, trigger: 'blur' }

    ],

    sex: [{ required: true, message: '请选择性别', trigger: 'blur' }],
    id_pic: [{ required: true, message: '请输入图片路径', trigger: 'blur' }],
    addr: [{ required: true, message: '请输入地址', trigger: 'blur' }],

})
// 添加按钮
let add_ = () => {
    getUserList()
    dialogVisible.value = true
    dialogTitle.value = '添加用户'
    Object.keys(ruleForm).forEach(item => {
        ruleForm[item] = ''
    })
}
// 添加的请求
const addUser = async () => {
    // let { data: res } = await proxy.$http.post('expert/add', ruleForm)
    // console.log(res);
}
// 修改按钮
const edit_ = (row: any) => {
    getUserList()
    ruleForm = row
    dialogVisible.value = true
    dialogTitle.value = '修改用户'
}
// 修改的请求
const editUser = async () => {
    getUserList()
    // let { data: res } = await proxy.$http.put('expert/update', ruleForm)
    // console.log(res);

}
// 用户详情
const detail_ = (row: any) => {
    getUserList()
    dialogVisible.value = true
    dialogTitle.value = '用户详情'
    ruleForm = row
}
// 删除
const delete_ = async (id: any) => {
    let { data: res } = await proxy.$http.delete(`user?id=${id}`)
    if (res.code !== 200) {
        ElMessage({
            message: '删除失败',
            type: 'error',
            duration: 500
        })
    } else {
        ElMessage({
            message: '删除成功',
            type: 'success',
            duration: 500
        })
        getUserList()
    }

}
// 关闭对话框
const dialogClose = () => {
    ruleFormRef.value.resetFields()
}
// 点击对话框确定按钮
const dialogEnter = () => {
    ruleFormRef.value.validate((valid) => {
        if (!valid) {
            ElMessage({
                message: '请填写表单正确内容',
                type: 'error',
                duration: 500
            })
        } else {
            dialogVisible.value = false
            if (dialogTitle.value == '添加用户') {
                addUser()
            } if (dialogTitle.value == '修改用户') {
                editUser()
            }
        }
    })
}



onMounted(() => {
    getUserList()

})
</script>

<style lang="less" scoped></style>