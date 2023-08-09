<template>
    <div class="">
        <Bread></Bread>
        <el-card class="box-card">
            <template #header>
                <div class="card-header">
                    <el-button class="white" color="#13b9c9" @click="add_">添加专家</el-button>
                </div>
            </template>
            <el-table :data="expertList" border stripe style="width: 100%" show-overflow-tooltip :header-cell-style="{
                backgroundColor: 'rgba(19, 185, 201, 0.7)',
                color: 'rgb(70, 69, 69)'
            }">
                <el-table-column prop="name" label="姓名" width="" />
                <el-table-column prop="title" label="职称" width="" />
                <el-table-column prop="hospital_title" label="所属医院" width="200px" />
                <el-table-column prop="office_title" label="科室" />
                <el-table-column prop="goodat" label="擅长诊治" />
                <el-table-column prop="profile" label="详细描述" width="350" />
                <el-table-column label="分类操作" width="380">
                    <template #default="scope">
                        <el-button class="white" color="#13b9c9" @click="edit_(scope.row)">修改专家</el-button>
                        <el-button class="white" color="#13b9c9" @click="detail_(scope.row)">专家详情</el-button>
                        <el-popconfirm title="确定删除吗?" @confirm="delete_(scope.row.id)">
                            <template #reference>
                                <el-button class="white" color="#13b9c9">删除专家</el-button>
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
                    <el-input v-model="ruleForm.name" v-if="dialogTitle !== '专家详情'" />
                    <span v-else>{{ ruleForm.name }}</span>
                </el-form-item>
                <el-form-item label="所属医院" prop="hospital_title">
                    <el-select v-model="ruleForm.hospital_title" clearable placeholder="请选择" v-if="dialogTitle !== '专家详情'">
                        <el-option label="郑州第一附属医院" value="郑州第一附属医院" />
                        <el-option label="乐清市人民医院" value="乐清市人民医院" />
                        <el-option label="北京安贞医院" value="北京安贞医院" />
                        <el-option label="中国人民解放军总医院" value="中国人民解放军总医院" />
                        <el-option label="北京协和医院" value="北京协和医院" />
                        <el-option label="郑州大学附属郑州中心医院" value="郑州大学附属郑州中心医院" />
                        <el-option label="湘雅医院" value="湘雅医院" />
                        <el-option label="承德县医院" value="承德县医院" />
                    </el-select>
                    <span v-else>{{ ruleForm.hospital_title }}</span>
                </el-form-item>
                <el-form-item label="所属科室" prop="office_title">
                    <el-select v-model="ruleForm.office_title" clearable placeholder="请选择" v-if="dialogTitle !== '专家详情'">
                        <el-option label="心内科" value="心内科" />
                        <el-option label="耳鼻喉科" value="耳鼻喉科" />
                        <el-option label="呼吸内科" value="呼吸内科" />
                        <el-option label="外科" value="外科" />
                        <el-option label="儿科" value="儿科" />
                        <el-option label="妇产科" value="妇产科" />
                        <el-option label="男科" value="男科" />
                        <el-option label="口腔科" value="口腔科" />
                    </el-select>
                    <span v-else>{{ ruleForm.office_title }}</span>
                </el-form-item>
                <el-form-item label="职称" prop="title">
                    <el-select v-model="ruleForm.title" clearable placeholder="请选择" v-if="dialogTitle !== '专家详情'">
                        <el-option label="医师" value="医师" />
                        <el-option label="主治医师" value="主治医师" />
                        <el-option label="副主任医师" value="副主任医师" />
                        <el-option label="主任医师" value="主任医师" />
                    </el-select>
                    <span v-else>{{ ruleForm.title }}</span>
                </el-form-item>
                <el-form-item label="照片" prop="id_pic">
                    <el-input v-model="ruleForm.id_pic" v-if="dialogTitle !== '专家详情'" />
                    <span v-else><img :src="ruleForm.id_pic" alt=""></span>
                </el-form-item>
                <el-form-item label="擅长诊治" prop="goodat">
                    <el-input v-model="ruleForm.goodat" type="textarea" v-if="dialogTitle !== '专家详情'" />
                    <span v-else v-html="ruleForm.goodat"></span>
                </el-form-item>
                <el-form-item label="详情描述" prop="profile">
                    <el-input v-model="ruleForm.profile" type="textarea" v-if="dialogTitle !== '专家详情'" />
                    <span v-else v-html="ruleForm.profile"></span>
                </el-form-item>
            </el-form>
            <template #footer>
                <span class="dialog-footer" v-if="dialogTitle !== '专家详情'">
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
let expertList = ref([])
let pno = ref(1)
let count = ref(9)
let total = ref(10)
const getExpertList = async () => {
    let { data: res } = await proxy.$http.get(`expert/list?pno=${pno.value}&count=${count.value}`)
    if (res.code !== 200) {
        ElMessage({
            message: '请求失败',
            type: 'error',
            duration: 500
        })
    } else {
        expertList.value = res.data
        total.value = res.total
        pno.value = res.pno
    }
}
// 分页器
const handleSizeChange = (val: any) => {
    count.value = val
    getExpertList()
}
const handleCurrentChange = (val: any) => {
    pno.value = val
    getExpertList()
}
// dialog
let dialogVisible = ref(false)
const dialogTitle = ref('')
// from表单
let ruleFormRef = ref<FormInstance>()
let ruleForm = reactive({
    name: '',
    hospital_title: '',
    office_title: '',
    title: '',
    id_pic: '',
    goodat: '',
    profile: ''
})
let rules = reactive<FormRules>({
    name: [{ required: true, message: '请输入姓名', trigger: 'blur' }],
    hospital_title: [{ required: true, message: '请选择所属医院', trigger: 'blur' }],
    office_title: [{ required: true, message: '请选择所属科室', trigger: 'blur' }],
    title: [{ required: true, message: '请选择职称', trigger: 'blur' }],
    id_pic: [{ required: true, message: '请输入图片路径', trigger: 'blur' }],
    goodat: [{ required: true, message: '请输入擅长诊治', trigger: 'blur' }],
    profile: [{ required: true, message: '请输入详情描述', trigger: 'blur' }],
})
// 添加按钮
let add_ = () => {
    getExpertList()
    dialogVisible.value = true
    dialogTitle.value = '添加专家'
    Object.keys(ruleForm).forEach(item => {
        ruleForm[item] = ''
    })
}
// 添加的请求
const addExpert = async () => {
    let { data: res } = await proxy.$http.post('expert/add', ruleForm)
    // console.log(res);
}
// 修改按钮
const edit_ = (row: any) => {
    getExpertList()
    ruleForm = row
    dialogVisible.value = true
    dialogTitle.value = '修改专家'
}
// 修改的请求
const editExpert = async () => {
    let { data: res } = await proxy.$http.put('expert/update', ruleForm)
    // console.log(res);

}
// 专家详情
const detail_ = (row: any) => {
    dialogVisible.value = true
    dialogTitle.value = '专家详情'
    ruleForm = row
}
// 删除
const delete_ = async (id: any) => {
    let { data: res } = await proxy.$http.delete(`expert?id=${id}`)
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
        getExpertList()
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
            if (dialogTitle.value == '添加专家') {
                addExpert()
            } if (dialogTitle.value == '修改专家') {
                editExpert()
            }
        }
    })
}



onMounted(() => {
    getExpertList()

})
</script>

<style lang="less" scoped></style>