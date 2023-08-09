<template>
    <div class="">
        <Bread></Bread>
        <el-card class="box-card">
            <template #header>
                <div class="card-header">
                    <el-button class="white" color="#13b9c9" @click="add_">添加处方</el-button>
                </div>
            </template>
            <el-table :data="recipeList" border stripe style="width: 100%" show-overflow-tooltip :header-cell-style="{
                backgroundColor: 'rgba(19, 185, 201, 0.7)',
                color: 'rgb(70, 69, 69)'
            }">
                <el-table-column prop="doctor" label="主治医生" width="" />
                <el-table-column prop="type" label="药品类型" width="" />
                <el-table-column prop="result" label="适用症状" width="200px" />
                <el-table-column prop="frequency" label="使用方法" />
                <el-table-column prop="oid" label="处方编号" />

                <el-table-column label="分类操作" width="380">
                    <template #default="scope">
                        <el-button class="white" color="#13b9c9" @click="edit_(scope.row)">修改处方</el-button>
                        <el-button class="white" color="#13b9c9" @click="detail_(scope.row)">处方详情</el-button>
                        <el-popconfirm type="确定删除吗?" @confirm="delete_(scope.row.id)">
                            <template #reference>
                                <el-button class="white" color="#13b9c9">删除处方</el-button>
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
                <el-form-item label="主治医生" prop="doctor">
                    <el-input v-model="ruleForm.doctor" v-if="dialogTitle !== '处方详情'" />
                    <span v-else>{{ ruleForm.doctor }}</span>
                </el-form-item>
                <el-form-item label="药品类型" prop="type">
                    <el-select v-model="ruleForm.type" clearable placeholder="请选择" v-if="dialogTitle !== '处方详情'">
                        <el-option label="中药" value="中药" />
                        <el-option label="西药" value="西药" />
                        <el-option label="中成药" value="中成药" />
                        <el-option label="外用药" value="外用药" />

                    </el-select>
                    <span v-else>{{ ruleForm.type }}</span>
                </el-form-item>
                <el-form-item label="适用症状" prop="result">
                    <el-select v-model="ruleForm.result" clearable placeholder="请选择" v-if="dialogTitle !== '处方详情'">
                        <el-option label="头疼" value="头疼" />
                        <el-option label="肚子疼" value="肚子疼" />
                        <el-option label="发烧" value="发烧" />
                        <el-option label="急性肠炎" value="急性肠炎" />

                    </el-select>
                    <span v-else>{{ ruleForm.result }}</span>
                </el-form-item>
                <el-form-item label="使用方法" prop="frequency">

                    <el-input v-model="ruleForm.frequency" v-if="dialogTitle !== '处方详情'" />
                    <span v-else>{{ ruleForm.frequency }}</span>
                </el-form-item>
                <el-form-item label="处方编号" prop="oid">
                    <el-input v-model="ruleForm.oid" type="textarea" v-if="dialogTitle !== '处方详情'" />
                    <span v-else v-html="ruleForm.oid"></span>
                </el-form-item>

            </el-form>
            <template #footer>
                <span class="dialog-footer" v-if="dialogTitle !== '处方详情'">
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
let recipeList = ref([])
let pno = ref(1)
let count = ref(3)
let total = ref(10)
const getRicipeList = async () => {
    let { data: res } = await proxy.$http.get(`pres/list?pno=${pno.value}&count=${count.value}`)
    // console.log(res);

    if (res.code !== 200) {
        ElMessage({
            message: '请求失败',
            type: 'error',
            duration: 500
        })
    } else {
        recipeList.value = res.data
        total.value = res.total
        pno.value = res.pno
    }
}
// 分页器
const handleSizeChange = (val: any) => {
    count.value = val
    getRicipeList()
}
const handleCurrentChange = (val: any) => {
    pno.value = val
    getRicipeList()
}
// dialog
let dialogVisible = ref(false)
const dialogTitle = ref('')
// from表单
let ruleFormRef = ref<FormInstance>()
let ruleForm = reactive({
    doctor: '',
    result: '',
    frequency: '',
    type: '',
    id_pic: '',
    oid: '',

})
let rules = reactive<FormRules>({
    doctor: [{ required: true, message: '请输入姓名', trigger: 'blur' }],
    result: [{ required: true, message: '请选择所属医院', trigger: 'blur' }],
    frequency: [{ required: true, message: '请选择所属科室', trigger: 'blur' }],
    type: [{ required: true, message: '请选择职称', trigger: 'blur' }],
    id_pic: [{ required: true, message: '请输入图片路径', trigger: 'blur' }],
    oid: [{ required: true, message: '请输入擅长诊治', trigger: 'blur' }],

})
// 添加按钮
let add_ = () => {
    getRicipeList()
    dialogVisible.value = true
    dialogTitle.value = '添加处方'
    Object.keys(ruleForm).forEach(item => {
        ruleForm[item] = ''
    })
}
// 添加的请求
const addExpert = async () => {
    // let { data: res } = await proxy.$http.post('pres/verify', ruleForm)
    // console.log(res);
}
// 修改按钮
const edit_ = (row: any) => {
    getRicipeList()
    ruleForm = row
    dialogVisible.value = true
    dialogTitle.value = '修改处方'
}
// 修改的请求
const editExpert = async () => {
    let { data: res } = await proxy.$http.put('pres/verify', ruleForm)
    // console.log(res);
    if (res.code !== 200) {
        ElMessage({
            message: '修改失败',
            type: 'error',
            duration: 500
        })
    } else {
        ElMessage({
            message: '修改成功',
            type: 'success',
            duration: 500
        })
        getRicipeList()
    }
}
// 处方详情
const detail_ = (row: any) => {
    getRicipeList()
    dialogVisible.value = true
    dialogTitle.value = '处方详情'
    ruleForm = row
}
// 删除
const delete_ = async (id: any) => {
    // let { data: res } = await proxy.$http.delete(`pres?id=${id}`)
    // if (res.code !== 200) {
    //     ElMessage({
    //         message: '删除失败',
    //         type: 'error',
    //         duration: 500
    //     })
    // } else {
    //     ElMessage({
    //         message: '删除成功',
    //         type: 'success',
    //         duration: 500
    //     })
    //     getRicipeList()
    // }

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
            if (dialogTitle.value == '添加处方') {
                addExpert()
            } if (dialogTitle.value == '修改处方') {
                editExpert()
            }
        }
    })
}



onMounted(() => {
    getRicipeList()

})
</script>

<style lang="less" scoped></style>