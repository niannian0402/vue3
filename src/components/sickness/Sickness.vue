<template>
    <div class="">
        <Bread></Bread>
        <el-card class="box-card">
            <template #header>
                <div class="card-header">
                    <el-button color="#13b9c9" class="white" @click="add_">添加分类</el-button>
                </div>
            </template>
            <!-- 表格区域 -->
            <el-table :data="sicknessList" stripe style="width: 100%" border :header-cell-style="{
                backgroundColor: 'rgba(19, 185, 201, 0.7)',
                color: 'rgb(70, 69, 69)'
            }" show-overflow-tooltip
                :tooltip-options="{ effect: 'dark', enterable: false, placement: 'top', showArrow: true, width: '80px' }">
                <el-table-column prop="title" label="疾病名称" width="150" />
                <el-table-column prop="profile" label="疾病描述" />
                <el-table-column prop="symptom" label="症状" />
                <el-table-column prop="diagnose" label="诊断" />
                <el-table-column label="分类操作" width="250px">
                    <template #default="scope">
                        <el-button color="#13b9c9" class="white" @click="edit_(scope.row)">修改疾病</el-button>
                        <el-popconfirm title="确定删除吗?" @confirm="delete_(scope.row.did)">
                            <template #reference>
                                <el-button color="#13b9c9" class="white">删除疾病</el-button>
                            </template>
                        </el-popconfirm>

                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页器 -->
            <el-pagination v-model:current-page="pno" v-model:page-size="count" :page-sizes="[3, 5, 8, 10]"
                layout="total, sizes, prev, pager, next, jumper" :total="total" @size-change="handleSizeChange"
                @current-change="handleCurrentChange" />
        </el-card>
        <!-- 对话框 -->
        <el-dialog v-model="dialogVisible" :title="dialogTitle" width="40%" @close="dialogClose">
            <el-form ref="ruleFormRef" :model="ruleForm" :rules="rules" label-width="120px" status-icon>
                <el-form-item label="疾病名称" prop="title">
                    <el-input v-model="ruleForm.title" />
                </el-form-item>
                <el-form-item label="疾病描述" prop="profile">
                    <el-input v-model="ruleForm.profile" />
                </el-form-item>
                <el-form-item label="症状" prop="symptom">
                    <el-input v-model="ruleForm.symptom" />
                </el-form-item>
                <el-form-item label="诊断" prop="diagnose">
                    <el-input v-model="ruleForm.diagnose" />
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
import { ref, reactive, getCurrentInstance, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import type { FormInstance, FormRules } from 'element-plus'
let { proxy }: any = getCurrentInstance()
let sicknessList = ref([])
let pno = ref(1)
let count = ref(10)
let total = ref(10)
const getSicknessList = async () => {
    // let { data: res } = await proxy.$http.get("disease/list")
    let { data: res } = await proxy.$http.get(`disease/list?count=${count.value}&pno=${pno.value}`)
    // console.log(res);

    if (res.code !== 200) {
        ElMessage({
            message: '请求疾病列表失败',
            type: 'error',
            duration: 500
        })
    } else {
        sicknessList.value = res.data
        pno.value = res.pno
        // count.value = res.count
        total.value = res.total
    }

}
let dialogTitle = ref('')
let dialogVisible = ref(false)
let add_ = () => {
    dialogVisible.value = true
    dialogTitle.value = '添加疾病'
    getSicknessList()
    Object.keys(ruleForm).forEach(item => {
        ruleForm[item] = ''
    })
}
// from表单
let ruleForm = reactive({
    title: '',
    profile: '',
    symptom: '',
    diagnose: '',
    mtime: '',
    did: ''
})
const ruleFormRef = ref<FormInstance>()
const rules = reactive<FormRules>({
    title: [{ required: true, message: '请填写疾病名称', trigger: 'blur' }],
    profile: [{ required: true, message: '请填写疾病描述', trigger: 'blur' }],
    symptom: [{ required: true, message: '请填写症状', trigger: 'blur' }],
    diagnose: [{ required: true, message: '请填写诊断', trigger: 'blur' }]
})
// 对话框关闭事件
const dialogClose = () => {
    ruleFormRef.value.resetFields()
}
// 发起添加疾病的请求
const addSickness = async () => {
    let { data: res } = await proxy.$http.post('disease/add', ruleForm)
    if (res.code !== 200) {
        ElMessage({
            message: '添加疾病失败',
            type: 'error',
            duration: 500
        })
    } else {
        ElMessage({
            message: '添加疾病成功',
            type: 'success',
            duration: 500,
            onClose: () => {
                dialogVisible.value = false;

                getSicknessList()
            }
        })


    }

}
// 修改疾病
const edit_ = (row: any) => {
    dialogVisible.value = true
    dialogTitle.value = '修改疾病'
    ruleForm = row
    getSicknessList()
}
// 修改疾病的请求
const eidtSickness = async () => {
    let { data: res } = await proxy.$http.put('disease/update', ruleForm)
    if (res.code !== 200) {
        ElMessage({
            message: '修改疾病失败',
            type: 'error',
            duration: 500
        })
    } else {
        ElMessage({
            message: '修改疾病成功',
            type: 'success',
            duration: 500,
            onClose: () => {
                dialogVisible.value = false;
                getSicknessList()
            }
        })

    }
}
// 删除
let delete_ = async (did: any) => {
    let { data: res } = await proxy.$http.delete(`disease?did=${did}`)
    if (res.code !== 200) {
        ElMessage({
            message: '删除疾病失败',
            type: 'error',
            duration: 500
        })
    } else {
        ElMessage({
            message: '删除疾病成功',
            type: 'success',
            duration: 500,
        })
        getSicknessList()
    }
}
// 对话框点击确定按钮
const dialogEnter = () => {
    // 对表单的预验证
    ruleFormRef.value.validate((valid) => {
        if (!valid) {
            ElMessage({
                message: '请输入内容',
                type: 'error',
                duration: 500
            })
        } else {
            if (dialogTitle.value == '添加疾病') {
                // 发起添加的请求
                addSickness()
            }
            if (dialogTitle.value == '修改疾病') {
                // 发起修改疾病的请求
                eidtSickness()
            }

        }
    })
}
// 分页器
let handleSizeChange = (val: any) => {
    count.value = val
    getSicknessList()
}
let handleCurrentChange = (val: any) => {
    pno.value = val
    getSicknessList()
}
onMounted(() => {
    getSicknessList()
})
</script>

<style lang="less" scoped></style>