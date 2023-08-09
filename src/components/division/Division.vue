<template>
    <div class="">
        <Bread></Bread>
        <el-card class="box-card">
            <template #header>
                <div class="card-header">

                    <el-button color="#13b9c9" class="white" @click="add_">添加科室</el-button>
                </div>
            </template>
            <!-- 表格区域 -->
            <el-table :data="divisionList" style="width: 100%" border :header-cell-style="{
                backgroundColor: 'rgba(19, 185, 201, 0.7)',
                color: 'rgb(70, 69, 69)'
            }">
                <el-table-column prop="title" label="科室名称" />
                <el-table-column prop="pic" label="图片">
                    <template #default="scope">
                        <img class="piccs" :src="scope.row.pic" alt="">
                    </template>
                </el-table-column>
                <el-table-column label="分类操作">
                    <template #default="scope">
                        <el-button color="#13b9c9" class="white" @click="edit_(scope.row)">修改科室</el-button>

                        <el-popconfirm title="确定删除吗?" @confirm="delete_(scope.row.id)">
                            <template #reference>
                                <el-button color="#13b9c9" class="white">删除科室</el-button>
                            </template>
                        </el-popconfirm>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页器 -->
            <el-pagination v-model:current-page="pno" v-model:page-size="count" :page-sizes="[3, 5, 8, 12]"
                layout="total, sizes, prev, pager, next, jumper" :total="total" @size-change="handleSizeChange"
                @current-change="handleCurrentChange" />
        </el-card>
        <!-- 对话框 -->
        <el-dialog v-model="dialogVisible" :title="dialogTitle" width="70%" @close="dialogClose">
            <el-form ref="ruleFormRef" :model="ruleForm" :rules="rules" label-width="120px" class="demo-ruleForm"
                status-icon>
                <el-form-item label="科室名称" prop="title">
                    <el-input v-model="ruleForm.title" />
                </el-form-item>
                <el-form-item label="图片" prop="pic">
                    <el-input v-model="ruleForm.pic" />
                </el-form-item>
            </el-form>
            <template #footer>
                <span class="dialog-footer">
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
import type { FormInstance, FormRules } from 'element-plus'
import { ElMessage } from 'element-plus'
import { ref, reactive, getCurrentInstance, onMounted } from 'vue'
let { proxy }: any = getCurrentInstance()
let divisionList = ref([])
let count = ref(5)
let pno = ref(1)
let total = ref(10)
const getDivsionList = async () => {
    let { data: res } = await proxy.$http.get(`office/list?pno=${pno.value}&count=${count.value}`)
    if (res.code !== 200) {
        ElMessage(
            {
                type: 'error',
                message: '请求失败',
                duration: 500
            }
        )
    } else {
        divisionList.value = res.data
        pno.value = res.pno
        total.value = res.total
    }
}
// 分页器
const handleSizeChange = (val: any) => {
    count.value = val
    getDivsionList()
}
const handleCurrentChange = (val: any) => {
    pno.value = val
    getDivsionList()
}
let dialogVisible = ref(false)
let dialogTitle = ref('')
let add_ = () => {
    dialogVisible.value = true
    dialogTitle.value = '添加科室'
    Object.keys(ruleForm).forEach(item => {
        ruleForm[item] = ''
    })
    getDivsionList()
}
// 表单数据
const ruleFormRef = ref<FormInstance>()
let ruleForm = reactive({
    title: '',
    pic: ''
})
const rules = reactive<FormRules>({
    title: [{ required: true, message: '请输入科室名称', trigger: 'blur' },],
    pic: [{ required: true, message: '请填写图片路径', trigger: 'blur' },],
})
let dialogClose = () => {
    ruleFormRef.value.resetFields()
}
let addDivision = async () => {
    let { data: res } = await proxy.$http.post('office/add', ruleForm)
    if (res.code !== 200) {
        ElMessage(
            {
                type: 'error',
                message: '添加失败',
                duration: 500
            }
        )
    } else {
        ElMessage(
            {
                type: 'success',
                message: '添加成功',
                duration: 500,
                onClose: () => {
                    dialogVisible.value = false
                    getDivsionList()
                }
            }
        )

    }

}
// 修改科室
let edit_ = (row: any) => {
    getDivsionList()
    ruleForm = row
    dialogVisible.value = true
    dialogTitle.value = '修改科室'
}
let editDivision = async () => {
    let { data: res } = await proxy.$http.put('office/update', ruleForm)
    if (res.code !== 200) {
        ElMessage({
            type: 'error',
            message: '修改失败',
            duration: 500
        })
    } else {
        ElMessage({
            type: 'success',
            message: '修改科室成功',
            duration: 500,
            onClose: () => {
                dialogVisible.value = false
                getDivsionList()
            }
        })
    }

}
// 删除
let delete_ = async (id: any) => {
    let { data: res } = await proxy.$http.delete(`office?id=${id}`)
    if (res.code !== 200) {
        ElMessage({
            type: 'error',
            message: '删除失败',
            duration: 500
        })
    } else {
        ElMessage({
            message: '删除科室成功',
            type: 'success',
            duration: 500
        })
        getDivsionList()
    }
}
let dialogEnter = () => {
    ruleFormRef.value.validate((valid) => {
        if (!valid) {
            ElMessage({
                message: '请填写表单内容',
                type: 'error',
                duration: 500
            })
        } else {
            if (dialogTitle.value == '添加科室') {
                addDivision()
            } if (dialogTitle.value == '修改科室') {
                editDivision()
            }
        }
    })

}
onMounted(() => {
    getDivsionList()
})
</script>

<style lang="less" scoped>
.el-table {
    height: 500px !important;
}

.piccs {
    width: 100px !important;
}
</style>