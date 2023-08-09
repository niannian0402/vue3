<template>
    <div class="">
        <Bread></Bread>
        <el-card class="box-card">
            <template #header>
                <div class="card-header">
                    <el-button class="white" color="#13b9c9" @click="add_">添加广告</el-button>
                </div>
            </template>
            <el-table :data="ADList" border stripe style="width: 100%" show-overflow-tooltip :header-cell-style="{
                backgroundColor: 'rgba(19, 185, 201, 0.7)',
                color: 'rgb(70, 69, 69)'
            }">
                <el-table-column prop="client" label="广告编号" width="" />
                <el-table-column prop="title" label="网址" width="" />
                <el-table-column prop="pic" label="广告牌图片" width="300px">
                    <template #default="scope">
                        <img class="adpic" :src="scope.row.pic" alt="">
                    </template>
                </el-table-column>
                <el-table-column prop="place" label="地址标号" />

                <el-table-column label="分类操作" width="380">
                    <template #default="scope">
                        <el-button class="white" color="#13b9c9" @click="edit_(scope.row)">修改广告</el-button>
                        <el-button class="white" color="#13b9c9" @click="detail_(scope.row)">广告详情</el-button>
                        <el-popconfirm title="确定删除吗?" @confirm="delete_(scope.row.id)">
                            <template #reference>
                                <el-button class="white" color="#13b9c9">删除广告</el-button>
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
        <el-dialog v-model="dialogVisible" :title="dialogtitle" width="60%" @close="dialogClose">
            <el-form ref="ruleFormRef" :model="ruleForm" :rules="rules" label-width="120px" status-icon>
                <el-form-item label="广告编号" prop="client">
                    <el-input v-model="ruleForm.client" v-if="dialogtitle !== '广告详情'" />
                    <span v-else>{{ ruleForm.client }}</span>
                </el-form-item>
                <el-form-item label="图片" prop="pic">
                    <el-input v-model="ruleForm.pic" v-if="dialogtitle !== '广告详情'" />

                    <span v-else><img class="pic" :src="ruleForm.pic" alt=""></span>


                </el-form-item>
                <el-form-item label="广告区域" prop="place">
                    <el-input v-model="ruleForm.place" v-if="dialogtitle !== '广告详情'" />
                    <span v-else>{{ ruleForm.place }}</span>
                </el-form-item>
                <el-form-item label="title" prop="title">
                    <el-input v-model="ruleForm.href" v-if="dialogtitle !== '广告详情'" />
                    <span v-else>{{ ruleForm.href }}</span>
                </el-form-item>

            </el-form>
            <template #footer>
                <span class="dialog-footer" v-if="dialogtitle !== '广告详情'">
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
let ADList = ref([])
let pno = ref(1)
let count = ref(9)
let total = ref(10)
const getADList = async () => {
    let { data: res } = await proxy.$http.get('ad/list')
    // console.log(res);

    if (res.code !== 200) {
        ElMessage({
            message: '请求失败',
            type: 'error',
            duration: 500
        })
    } else {
        ADList.value = res.data
        total.value = res.total
        pno.value = res.pno
    }
}
// 分页器
const handleSizeChange = (val: any) => {
    count.value = val
    getADList()
}
const handleCurrentChange = (val: any) => {
    pno.value = val
    getADList()
}
// dialog
let dialogVisible = ref(false)
const dialogtitle = ref('')
// from表单
let ruleFormRef = ref<FormInstance>()
let ruleForm = reactive({
    client: '',
    pic: '',
    place: '',
    href: '',

})
let rules = reactive<FormRules>({
    client: [{ required: true, message: '请输入广告编号', trigger: 'blur' }],
    pic: [{ required: true, message: '请选择广告图片', trigger: 'blur' }],
    place: [{ required: true, message: '请选择广告', trigger: 'blur' }],
    href: [{ required: true, message: '请输入网站', trigger: 'blur' }],

})
// 添加按钮
let add_ = () => {
    getADList()
    dialogVisible.value = true
    dialogtitle.value = '添加广告'
    Object.keys(ruleForm).forEach(item => {
        ruleForm[item] = ''
    })
}
// 添加的请求
const addAD = async () => {
    let { data: res } = await proxy.$http.post('ad/add', ruleForm)
    // console.log(res);
    if (res.code != 200) {
        ElMessage({
            message: '添加失败',
            type: 'error',
            duration: 500
        })
    } else {
        ElMessage({
            message: '添加成功',
            type: 'success',
            duration: 500
        })
        getADList()
        dialogVisible.value = false
    }
}
// 修改按钮
const edit_ = (row: any) => {
    getADList()
    ruleForm = row
    dialogVisible.value = true
    dialogtitle.value = '修改广告'
}
// 修改的请求
const editAD = async () => {
    let { data: res } = await proxy.$http.put('ad/update', ruleForm)
    if (res.code != 200) {
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
        getADList()
        dialogVisible.value = false
    }

}
// 广告详情
const detail_ = (row: any) => {
    getADList()
    dialogVisible.value = true
    dialogtitle.value = '广告详情'
    ruleForm = row
}
// 删除
const delete_ = async (id: any) => {
    let { data: res } = await proxy.$http.delete(`ad/del?id=${id}`)
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
        getADList()
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
            if (dialogtitle.value == '添加广告') {
                addAD()
            } if (dialogtitle.value == '修改广告') {
                editAD()
            }
        }
    })
}



onMounted(() => {
    getADList()

})
</script>

<style lang="less" scoped>
.adpic {
    width: 60px;
}
</style>